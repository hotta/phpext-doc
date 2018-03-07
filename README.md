[PHP Extension 開発入門](https://net-newbie.com/phpext/)のソース置き場

# 環境構築

[https://github.com/hotta/ansible-centos7](https://github.com/hotta/ansible-centos7) 相当のベース環境(CentOS 7.3 + α)
```bash
$ ansible-playbook /etc/ansible/jobs/sphinx.yml 
$ git clone git@github.com:hotta/phpext-doc.git
$ mkdir work
$ cd work
$ sphinx-quickstart
```
質問に答える。"Project name", "Author name(s)", "Project version" は必須。
後は [Enter] でOK。
```bash
$ cp -rp ~/phpext-doc/* .
$ make html
