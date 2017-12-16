---
layout: post
title: "Lendo e-mails do Gmail com Python 3"
description: "Usando Python para conectar no Gmail, extrair uma informação específica do corpo e adicionar um label"
category: ColdFusion/CFML
tags: [Python]
comments: true
---
Boas Pessoal!


Nesta semana tive uma necessidade inusitada e resolvi usar o Python para
me ajudar a resolver este problema.

#### O contexto
Precisava ler dentre 1000 e-mails do Gmail, quais tinham um determinado
texto, até aqui, o próprio Gmail resolvia, mas, eu ainda precisava extrair
3 informações específicas do corpo de cada um destes e-mails e após isto, atribuir um label específico para facilitar futuras buscas.

#### Mão na massa
Primeiro configuramos as constantes com os dados de acesso a conta. Para leitura do Gmail, utilizamos o servidor  imap.gmail.com.
Caso você utilize autenticação em 2 etapas, configure uma [“senha de app” no Gmail][] e utilize-a na constante `FROM_PWD`, caso contrário, terá falhas na autenticação.
```python
FROM_EMAIL = "<seuemail>@gmail.com"  # substitua <seuemail> pelo seu email.
FROM_PWD = "<suasenha>"  # substitua <suasenha> pela sua senha
SMTP_SERVER = "imap.gmail.com"  # padrão
SMTP_PORT = 993  # padrão
```


Utilizamos o módulo `imaplib` para conectar no Gmail via SSL e o módulo `email` para fazer o parser do e-mail.
```python
import imaplib
import email
```


Abrimos a conexão com o servidor e efetuamos o login.
```python
mail = imaplib.IMAP4_SSL(SMTP_SERVER)
mail.login(FROM_EMAIL, FROM_PWD)
```


Neste ponto selecionamos a caixa de e-mail que queremos fazer a leitura, no meu caso, utilizei “inbox” para leitura dos e-mails.
Como também vamos atribuir um label aos e-mails “processados”, passamos o parâmetro `readonly=False`.
```python
mail.select('inbox', readonly=False)
```


Agora filtramos os e-mails pelo assunto específico, utilizando o critério `SUBJECT`. Veja a [RFC 2060][] para saber sobre outros critérios de busca, como *recent*, *before*, *cc*, etc.
```python
type_mail, data = mail.search(None, 'SUBJECT ASSUNTO-ESPECÍFICO')
Desta forma, temos uma lista com os IDs dos e-mails.
```


Pegamos o primeiro e o último ID para iterarmos em cada um deles utilizando `range()`.
```python
mail_ids = data[0]

id_list = mail_ids.split()
first_email_id = int(id_list[0])
latest_email_id = int(id_list[-1])

for i in range(latest_email_id, first_email_id, -1):
```


Para obter os detalhes de cada e-mail, usamos `mail.fetch` com o protocolo RFC822.
```python
typ, data = mail.fetch(str.encode(str(i)), '(RFC822)')
```
Onde *i* é o ID de cada e-mail que estamos iterando.


E obtemos os detalhes do e-mail como assunto e remetente.
```python
msg = email.message_from_string(response_part[1].decode('utf-8'))
email_subject = msg['subject']
email_from = msg['from']
```

Para obter as informações específicas do corpo do e-mail, convertemos para string.
```python
mail_str = str(msg)
```

Então pegamos as informações específicas, utilizando *find* e *slice*. Talvez usar o [Beautiful Soup][] tornaria o parser menos verboso.
```python
# INFORMAÇÃO DE DETALHE
detail_pos_ini = mail_str.find('class="luceeH0">Detail</td>')
detail_pos_inter = mail_str.find('<td class="luceeN1">', detail_pos_ini)
detail_pos_fim = mail_str.find('</td>', detail_pos_inter)
detail_str = mail_str[detail_pos_inter + 20:detail_pos_fim]

# SQL EVENTO
sql_pos_ini = mail_str.find('<td class="luceeH0">SQL</td>')
sql_pos_inter = mail_str.find('<td class="luceeN1">', sql_pos_ini)
sql_pos_fim = mail_str.find('</td>', sql_pos_inter)
sql_str = mail_str[sql_pos_inter + 20:sql_pos_fim]

# DATA/HORA EVENTO
datetime_error_pos_ini = mail_str.find('<h2>{ts ')
datetime_error_pos_fim = mail_str.find('</h2>', datetime_error_pos_ini)
datetime_error = mail_str[datetime_error_pos_ini + 4:datetime_error_pos_fim]
```

Para a necessidade, fizemos prints gerando um arquivo com a saída da execução do script Python.
```python
print('DATA/HORA: ' + datetime_error + '\n')
print('EVENTO: ' + detail_str.replace('\n', ' ') + '\n')
print('SQL: \n' + sql_str + '\n')
```

E por fim, adicionamos um label ao e-mail que acabamos de iterar usando `mail.store`.
```python
labeled = mail.store(str.encode(str(i)), '+X-GM-LABELS', ’seu_label')
```
Caso queira remover o label, substitua o sinal ‘+’ por ‘-‘.`

Após iterar a lista de e-mails, efetuamos o logout no servidor.
```python
mail.logout()
```

É isto.

E você, já precisou conectar no servidor de e-mail com Python? Gostaria de acrescentar, corrigir ou sugerir algo?
Comente a vontade!

Gde. abraço!

Ed


#### Todos os links deste artigo


* [“senha de app” no Gmail][]
* [RFC 2060][]
* [Beautiful Soup][]



[“senha de app” no Gmail]: https://support.google.com/accounts/answer/185833?hl=pt-BR
[RFC 2060]: https://tools.ietf.org/html/rfc2060.html#section-6.4.4
[Beautiful Soup]: https://www.crummy.com/software/BeautifulSoup/bs4/doc/