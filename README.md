# Ansible roles lab for classrooms: a role, handlers and Vault on a VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml)

A ready-made **Ansible roles lab** for teaching infrastructure as code, from the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning), where schools and colleges give every student their own machines and terminal in the browser. It is the second level after [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics). Each student gets an Ubuntu **VM** with **ansible-core** installed and a small project, and builds one **role** from nothing: its variables, a Jinja2 template, tasks with tags, a **handler** that runs once, and a token kept in **Ansible Vault**. The machine itself is the target, so nothing else is needed and nothing is installed on the student's computer.

## What the student does

The lab leaves a project in `~/roles-lab` and a `README.md` in the home folder that says what to do in seven steps and gives none of the answers:

```
ansible.cfg          points at inventory.ini and the roles folder
inventory.ini        two hosts, web1 and spare1, both this machine
site.yml             one play on web that uses the role appconf
vault-password.txt   the vault password for this lab
roles/               where the role goes
```

1. Run the playbook and see it stop, because the role `appconf` does not exist yet.
2. Make the role with `ansible-galaxy role init`, and fill in its `meta/main.yml`.
3. Set variables in `defaults/main.yml` and `vars/main.yml`. `site.yml` already sets `app_env` and `app_dir` on the play, so the student has to find out which variable wins.
4. Write the tasks, tagged `config` or `user`: a folder, a file made from a template with a loop and a condition, one file per feature, a token file, a system user and two small files, one only when the user was really created.
5. Make one handler that three tasks notify run once, and make the role run its handlers part way through.
6. Keep `app_api_token` in a file made with `ansible-vault`, opened with the password file, and written in plain text nowhere else.
7. Run it twice (the second run must change nothing), then try `--check --diff`, `--tags` and `--limit`.

## How it is marked

The lab has a **check**, a script that marks a handed-in VM without the teacher reading it. It runs on a copy of the saved machine after a fresh start. It reads the role first: the standard folders, a filled-in `meta/main.yml`, at least six tasks in the role and not in the playbook, a handler, a loop, tags, and a template with a `for` loop and an `if`. It refuses `shell`, `command`, `raw` and `script` (and a few other ways round Ansible's modules), and a `pipe` lookup that runs a program. It opens the vault file with the lab's password and refuses a token written in plain text in the project.

Then it **runs the student's own `site.yml` seven times with values nobody has seen**: on a clean machine, again with nothing left to change, as a dry run that must show the change and write nothing, with other values, with each tag, and with `--limit` on the host the play does not use. It counts the handler runs in the real output, so a handler that runs three times is refused, and reads the files that came out. The first line of what it prints says what is not right.

What it does not do: it never restarts a real service, since a check cannot prove that, and a student with root on their own VM could still tamper with Ansible itself. It is a check for a classroom, not a proof.

## What is in the file

`ansible-roles.lab.yaml` is a whole **VM lab definition**, exactly what the platform runs:

| Key | What it is |
| --- | --- |
| `cpus`, `memory`, `disk` | the size of each student's VM: 1 CPU, 1024 MB and 10 GB |
| `open` | the ports that open in the browser: none, the lab is the terminal |
| `setup` | a script run once, as root, the first time each student's VM starts: it installs `ansible-core` from Ubuntu's archive and writes the project, the playbook, a fake vault password and the README |
| `check` | the marking script, run as root on a copy of the saved disk after a fresh boot; its exit status is the result and `0` passes |

The vault password in the project is a **fixture**: it says it is not a secret, and the token the student vaults is demo data, never a real secret.

## Using it on the platform

**To set it as work for a class:**

1. In your module, add a task and choose the **Ubuntu VM lab** card.
2. Press the Ansible roles sample, or paste the contents of `ansible-roles.lab.yaml`. Using a sample replaces the text, and Undo brings yours back.
3. Publish and try your own copy first. Every student gets a VM of their own, kept between sessions.
4. Assign it like any task. When it is handed in, run the check on a student's row in the class list: it starts a copy of their saved machine that is not saved back, and shows whether it passed and what it printed.

A Basics lesson on the platform works in this lab. The help manual's [page for this lab](https://mitsurutech.co.uk/help/lab-ansible-roles), its [guide to Ubuntu VM labs](https://mitsurutech.co.uk/help/labs) and its [sample labs page](https://mitsurutech.co.uk/help/sample-labs) cover the rest.

No college account yet? [Your first year is free](https://mitsurutech.co.uk/pricing) for up to 50 students, and [signing up](https://mitsurutech.co.uk/signup) takes an email address.

## Checking the file

On every push, `.github/workflows/check.yml` checks that the file is valid YAML with the keys above and that `setup` and `check` are each inside the platform's 16 KB limit. It does not boot a VM: that is done on the platform's own machines.

## What does not work in a lab

The VM reaches Ubuntu's package archive, PyPI, npm and GitHub, and nothing else. The lab uses only Ubuntu's archive, so every job is done with Ansible's own modules on files, folders and a system user. It has no service to restart: the handler is shown by what a check can see, how many times it ran and a file it writes. The target is the VM itself, over a local connection.

## About

Made by [Mitsuru Technologies](https://mitsurutech.co.uk) for the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning): Python, Java, Go, Node, SQL and Linux in the browser for schools and colleges, with container labs, VM labs, web apps deployed from GitHub, and quizzes that mark themselves. Try code with no account in the [playground](https://mitsurutech.co.uk/playground).

More labs from the same place: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-sdn-lab](https://github.com/mitsurutech/lab-sdn-lab), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab) and [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licence

This work is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). You may share it and adapt it for learning and teaching, if you credit Mitsuru Technologies Ltd, do not use it commercially, and share anything you make from it under the same licence. The full text is in the `LICENSE` file.

[Ansible](https://www.ansible.com) (`ansible-core`, GPL-3.0) is installed from Ubuntu's archive when a VM starts; nothing of it is distributed here.
