# Emote Hub — GitHub Pages + Square Cloud

Galeria estática em português usando o catálogo `emotes.json`, as imagens `https://wipimagemitem.squareweb.app/image/{id}` e a API `https://emote-wip.squareweb.app/emote`.

## Publicar no GitHub Pages

1. Crie ou abra o repositório GitHub onde ficará a interface.
2. Envie `index.html` e `emotes.json` para a raiz do repositório.
3. No GitHub, abra **Settings → Pages** e escolha a branch e a pasta raiz (`/`).
4. Aguarde a publicação e abra a URL informada pelo GitHub Pages.

O site não precisa de build nem de dependências. `index.html` e `emotes.json` precisam permanecer na mesma pasta.

## Como o pedido é enviado

Ao selecionar um emote e preencher os campos, o navegador faz um GET no formato:

```text
https://emote-wip.squareweb.app/emote?tc=CODIGO_EQUIPE&uid1=UID_JOGADOR&emote_id=ID_DO_EMOTE
```

O código que gera os parâmetros fica no início do `<script>` em `index.html`. A página não pede senha de jogo.

## CORS na API

Como GitHub Pages e Square Cloud são origens diferentes, o servidor Flask da API deve enviar `Access-Control-Allow-Origin` permitindo **a origem exata** do seu site, por exemplo `https://SEU-USUARIO.github.io` ou `https://SEU-USUARIO.github.io/SEU-REPOSITORIO` (o cabeçalho Origin não inclui caminho; use apenas `https://SEU-USUARIO.github.io`). Se usar domínio próprio, permita esse domínio também.

Exemplo para adicionar ao Flask da API, logo após `app = Flask(__name__)`:

```python
ALLOWED_ORIGINS = {
    "https://SEU-USUARIO.github.io",
    # "https://seu-dominio.com",
}

@app.after_request
def add_cors_headers(response):
    origin = request.headers.get("Origin", "")
    if origin in ALLOWED_ORIGINS:
        response.headers["Access-Control-Allow-Origin"] = origin
        response.headers["Vary"] = "Origin"
    return response
```

Troque o exemplo pela origem real do seu GitHub Pages, salve, faça deploy da API e depois teste pelo próprio site. A página usa um GET simples sem cabeçalhos customizados, mas ainda precisa que a resposta inclua o cabeçalho CORS. CORS **não é autenticação** e não impede chamadas diretas à API.

## Segurança e limites

- A API original aceita a chamada de emote sem autenticação. Se o domínio for divulgado, qualquer pessoa pode tentar chamá-la diretamente; implemente autenticação/limites no servidor antes de abrir o uso ao público.
- O navegador mostra os parâmetros da requisição; não inclua tokens secretos nem senhas de jogo no HTML/JavaScript.
- O site mostra a resposta imediata da API. Isso não comprova que a ação no jogo foi concluída.
