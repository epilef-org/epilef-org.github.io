---
layout: post
author: epilef
---

# Olá, mundo!
Primeiro post após configurar a publicação.

## Jekyll
No instalador do ruby pra Windows existe a opção de instalar um ambiente MSYS2, que eu já tinha,
então desmarquei a opção.

Tentando um `gem install jekyll bundler` recebi mensagens de sucesso e um aviso `MSYS2 could not be
found. Please run 'ridk install'`. Hum, pensei que o git bash fosse um MSYS2.

Mas sem problemas, prosseguimos com a instalação do MSYS2. Sempre tentei sobreviver apenas com o
Git Bash no Windows, faz tempo que não olho para outras alternativas. Talvez seja um momento
interessante para rever isso.

O tal `ridk` não está no meu PATH, nem vou procurar, só executei novamente o ruby installer. À
propósito, a versão do instalador que utilizei nessa situação foi o rubyinstaller-devkit-3.4.8-1.
Finalmente chegando no final da instalação optei pelo `MSYS2 and MINGW development toolchain`.

Tentando novamente o `gem install jekyll bundler`, recebo umas mensagens de `Building native
extensions.` Acho que agora vai. E foi.

## "Dumb" Mirror com GitHub Actions

Eu queria fazer um tipo de mirror do meu repositório, mas não é um mirror, é só uma cópia das
alterações commitadas em outro repositório.

Inicialmente testei algumas actions de mirroring do Marketplace, mas não era exatamente o que eu
queria ou achei o setup muito complicado. Como tudo queria fazer era só copiar as alterações e
espelhar em outro repositório, um simples `cp -r` resolve. Isso é uma chave de Deploy registrada
para o repositório mirror.

Aí está o minha solução idiota:
- Duas actions/checkout@v4, uma para o repo original e outra para o mirror. Ambas especificando o
diretório de destino no parâmetro `path`.
- No checkout do mirror, é especificada a secret contento a chave com acesso de escrita para o repo
mirror. A privada é registrada nas secrets do presente repositório e a pública registrada no
mirror.
- As alterações copiadas para o mirror são incluídas em um commit e enviadas para o repositório.

```yaml
name: CI
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          path: private
      
      - name: Mirror Checkout
        uses: actions/checkout@v4
        with:
          ssh-key: ${{ secrets.NOREPLY_PUSH_KEY }}
          fetch-depth: 0
          path: mirror
          repository: owner/mirror-repo

      - name: Run a multi-line script
        run: |
          rm -rf private/.git
          rm -rf private/.github
          cp -r private/* mirror/
          ls private/ -lha
          ls mirror/ -lha
          cd mirror
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add .
          git commit -m "update mirror repo"
          git push
```

Esse malabarismo todo foi uma tentativa de anonimizar o autor das postagens em um repositório com
GitHub Pages. A ideia era que o post original fosse escrito usando uma conta pessoal em um
repositório privado e publicado pela action em um repositório público. Só que ainda é possível ver
publicamente o nome da conta resposável pelos deploymennts do GitHub Pages.

## Netlify

Procurando outra alternativa.