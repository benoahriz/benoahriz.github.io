---
title: Using Ansible to communicate with rest API
date: 2016-07-15 14:15:27
tags:
author:      "Benjamin Rizkowsky"
---

# Ansible to interact with a rest API
Sometimes you may want to interact with a rest api where an Ansible module doesn’t already exist. For those cases you may want to try using the uri module. Ansible already supports interacting with json so its pretty easy. Below I show an example of interacting with the Vault api. Link

You may want to use Vault in cases where there is more sensitive information that you need to ensure that it is encrypted as well as has an audit trail. Vault provides these things and they are fairly easy to setup.

This example assumes you have setup a few things first.

Things you need with links to the instructions.

1. [Github account with a personal access token](https://github.com/blog/1509-personal-api-tokens)

2. make sure you export the token to your environment variables or hardcode it. In this example I use the lookup Ansible plugin to look up the GITHUB_TOKEN env var. ```export GITHUB_TOKEN=xxxxxxxxxxxx```

3. [Vault setup to use Github as an authentication back end.](https://www.vaultproject.io/docs/auth/github.html)

Here we check out a Vault token using our github token.

``` bash
---
- name: ansible examples
  hosts: localhost
  vars:
    github_token: "{{ lookup('env','GITHUB_TOKEN') }}"
    home_directory: "{{ lookup('env','HOME') }}"
    user: "{{ lookup('env','USER') }}"
  tasks:
    - name: get vault token
      uri:
        url: "https://myvault.com:8200/v1/auth/github/login"
        method: POST
        validate_certs: no
        follow_redirects: all
        body:
          token: "{{ github_token }}"
        return_content: yes
        body_format: json
      register: token_output
      delegate_to: 127.0.0.1
    - debug: var=token_output.json.auth.client_token

```

Now that we have a Vault token we can write values to the keystore. Notice how you can embed json directly into the yaml :)

``` bash
#     vault returns a 204 on success not 200
    - name: use token to update a secret
      uri:
        url: "https://myvault.com:8200/v1/envs/ansible_examples"
        method: POST
        validate_certs: no
        follow_redirects: all
        HEADER_X-Vault-Token: "{{ token_output.json.auth.client_token }}"
        body: "{
                '{{ user }}': '{{ home_directory }}'
              }"
#        body:
#          user: "{{ home_directory }}"
        return_content: yes
        body_format: json
        status_code: 204
      register: add_secret
      delegate_to: 127.0.0.1
    - debug: var=add_secret

```

Here we can read back the encrypted key from the vault.

``` bash 

- name: use token to read a secret
  uri:
    url: "https://myvault.com:8200/v1/envs/ansible_examples"
    method: GET
    validate_certs: no
    follow_redirects: all
    HEADER_X-Vault-Token: "{{ token_output.json.auth.client_token }}"
    return_content: yes
    body_format: json
  register: read_secret
  delegate_to: 127.0.0.1

- debug: var=read_secret
- debug: "var=read_secret.json.data.{{ user }}"

```








