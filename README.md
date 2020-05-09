# MAIL
## Задание
установка почтового сервера  
1. Установить в виртуалке postfix+dovecot для приёма почты на виртуальный домен любым обсужденным на семинаре способом
2. Отправить почту телнетом с хоста на виртуалку
3. Принять почту на хост почтовым клиентом

Результат  
1. Полученное письмо со всеми заголовками
2. Конфиги postfix и dovecot
## Решение
```bash
vagrant up
```
запускает стенд настраиваемый плэйбуком [mail.yml](mail.yml)  
### Конфиг postfix
В конфиге postfix нужно указать 192.168.25.150 в **inet_interfaces** и подсети в **mynetworks**
```bash
[root@mail ~]# postconf -n
alias_database = hash:/etc/aliases
alias_maps = hash:/etc/aliases
command_directory = /usr/sbin
config_directory = /etc/postfix
daemon_directory = /usr/libexec/postfix
data_directory = /var/lib/postfix
debug_peer_level = 2
debugger_command = PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin ddd $daemon_directory/$process_name $process_id & sleep 5
html_directory = no
inet_interfaces = localhost, $myhostname, 192.168.25.150
inet_protocols = ipv4
mail_owner = postfix
mailq_path = /usr/bin/mailq.postfix
manpage_directory = /usr/share/man
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 192.168.0.0/16, 10.0.0.0/8, 127.0.0.0/8
newaliases_path = /usr/bin/newaliases.postfix
queue_directory = /var/spool/postfix
readme_directory = /usr/share/doc/postfix-2.10.1/README_FILES
sample_directory = /usr/share/doc/postfix-2.10.1/samples
sendmail_path = /usr/sbin/sendmail.postfix
setgid_group = postdrop
unknown_local_recipient_reject_code = 550
```
### конфиг dovecot
в конфиге dovecot  
путь к файлу почтового ящика  
**mail_location = mbox:~/mail:INBOX=/var/mail/%u**  
дать параметр mail_privileged_group для того чтоб Dovecot мог блокировать почтовые ящики (имел привилегии на это).  
**mail_privileged_group = mail**  
отключить ssl  
**ssl = no**  
включить авторизацию plaintext для упрощения доступа  
**disable_plaintext_auth = no**  
включение конфига с указанием файла с пользователями  
**!include auth-passwdfile.conf.ext**  
```bash
[root@mail ~]# doveconf -n
# 2.2.36 (1f10bfa63): /etc/dovecot/dovecot.conf
# OS: Linux 3.10.0-957.12.2.el7.x86_64 x86_64 CentOS Linux release 7.6.1810 (Core)  
# Hostname: mail.skynet.lan
disable_plaintext_auth = no
first_valid_uid = 1000
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_privileged_group = mail
mbox_write_locks = fcntl
namespace inbox {
  inbox = yes
  location = 
  mailbox Drafts {
    special_use = \Drafts
  }
  mailbox Junk {
    special_use = \Junk
  }
  mailbox Sent {
    special_use = \Sent
  }
  mailbox "Sent Messages" {
    special_use = \Sent
  }
  mailbox Trash {
    special_use = \Trash
  }
  prefix = 
}
passdb {
  driver = pam
}
passdb {
  args = scheme=CRYPT username_format=%u /etc/dovecot/users
  driver = passwd-file
}
ssl = no
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key =  # hidden, use -P to show it
userdb {
  driver = passwd
}
userdb {
  args = username_format=%u /etc/dovecot/users
  driver = passwd-file
}
```
## Проверка
### Отправка
```bash
[root@centos7 linux-dz-25]# telnet 192.168.25.150 25
Trying 192.168.25.150...
Connected to 192.168.25.150.
Escape character is '^]'.
220 mail.skynet.lan ESMTP Postfix
ehlo centos7
mail from: maxvogelfrei@gmail.com
rcpt to: vagrant@skynet.lan
data
Subject: TEST
TEST
123
.
quit250-mail.skynet.lan
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
250 2.1.0 Ok
250 2.1.5 Ok
354 End data with <CR><LF>.<CR><LF>
250 2.0.0 Ok: queued as 95F0A5322

221 2.0.0 Bye
Connection closed by foreign host.
```
### Получение
Прочитать письмо можно через telnet  
```bash
telnet 192.168.25.150 143
a login vagrant vagrant
a FETCH 1 (BODY.PEEK[HEADER])
a FETCH 1 (BODY[TEXT])
a logout
```
```bash
[root@centos7 linux-dz-25]# telnet 192.168.25.150 143
Trying 192.168.25.150...
Connected to 192.168.25.150.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE AUTH=PLAIN] Dovecot ready.
a login vagrant vagrant
a OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY SPECIAL-USE] Logged in
a select inbox
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 1 RECENT
* OK [UNSEEN 1] First unseen.
* OK [UIDVALIDITY 1589049846] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
a OK [READ-WRITE] Select completed (0.003 + 0.000 + 0.002 secs).
a FETCH 1 (BODY.PEEK[HEADER])
* 1 FETCH (BODY[HEADER] {295}
Return-Path: <maxvogelfrei@gmail.com>
X-Original-To: vagrant@skynet.lan
Delivered-To: vagrant@skynet.lan
Received: from centos7 (unknown [192.168.25.1])
	by mail.skynet.lan (Postfix) with ESMTP id 95F0A5322
	for <vagrant@skynet.lan>; Sat,  9 May 2020 18:43:22 +0000 (UTC)
Subject: TEST

)
a OK Fetch completed (0.001 + 0.000 secs).
a FETCH 1 (BODY[TEXT])
* 1 FETCH (FLAGS (\Seen \Recent) BODY[TEXT] {11}
TEST
123
)
a OK Fetch completed (0.001 + 0.000 secs).
a logout
* BYE Logging out
a OK Logout completed (0.001 + 0.000 secs).
Connection closed by foreign host.
```
