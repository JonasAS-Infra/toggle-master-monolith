# Evidência da API pública — 03/11/2026

Captura enviada pelo usuário na conversa, mostrando o terminal do Mac executando:

```text
curl -i http://18.220.243.88/flags/teste-aws
HTTP/1.1 200 OK
Server: nginx/1.30.6

{"is_enabled":false,"name":"teste-aws"}
```

No mesmo fluxo, a flag `teste-aws` foi criada com `is_enabled=true`, atualizada para `false` e consultada novamente depois que o serviço `togglemaster` ficou ativo. A imagem original permanece anexada à conversa. Sua cópia para esta pasta é opcional.
