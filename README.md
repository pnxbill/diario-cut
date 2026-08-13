# Diário do Cut

App de diário alimentar (kcal / proteína / fibra / água / peso) que roda como PWA
no GitHub Pages e guarda tudo em `dados.json` **neste mesmo repositório**.

`dados.json` é a fonte única de verdade. O app lê esse arquivo ao abrir e o
regrava a cada alteração, via API do GitHub. Nada fica preso em `localStorage`
de um sandbox — o Claude lê o mesmo arquivo pela URL raw e pode editá-lo.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | O app inteiro (HTML + CSS + JS, sem dependências) |
| `dados.json` | Os dados. Fonte única de verdade |
| `manifest.webmanifest` | Metadados do PWA |
| `sw.js` | Service worker — só a casca do app, nunca os dados |
| `icon-*.png` | Ícones da tela inicial |

## Instalação (uma vez)

1. Crie um repositório **público** chamado `diario-cut` e suba estes arquivos na raiz.
2. **Settings → Pages → Source: Deploy from a branch → `main` → `/ (root)` → Save.**
3. Espere ~1 minuto e abra `https://SEU-USUARIO.github.io/diario-cut/`.
4. No Safari: **Compartilhar → Adicionar à Tela de Início**.

O app descobre sozinho o usuário e o repositório pela URL do Pages, então a
**leitura** já funciona sem configurar nada. Para **gravar**, falta o token.

## Token (uma vez)

GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**

- **Repository access:** Only select repositories → `diario-cut`
- **Permissions → Repository permissions → Contents:** `Read and write`
- Expiração: escolha o prazo que preferir (dá para renovar depois)

Copie o token e cole no app em **Sincronização → Configuração do GitHub → Salvar e testar**.
O token fica só no `localStorage` do aparelho — nunca vai para o repositório.

## Como a sincronização funciona

- Toda alteração grava **na hora** no aparelho e agenda um envio ~2,5 s depois.
- Antes de gravar, o app relê o remoto e mescla: **por dia, vence o registro com
  `atualizado_em` mais recente.** Dias que só existem de um lado sempre sobrevivem.
- Sem internet: fica tudo salvo local, com aviso de pendência, e sobe sozinho
  quando a conexão voltar ou quando o app for reaberto.
- `⤓ Baixar dados.json` e `⤒ Carregar um dados.json` existem como saída de
  emergência se o GitHub estiver fora do ar.

## Editando o JSON na mão

Mexa só nestes lugares:

- `dias[].refeicoes[].itens[]` — os alimentos (`nome`, `kcal`, `proteina_g`, `fibra_g`)
- `dias[].peso_kg`, `dias[].agua_ml`, `dias[].tipo` (`run` ou `rest`)
- `meta.alvos` — as metas diárias, lidas pelo app (não são fixas no código)
- `meta.biblioteca` e `meta.rapidos` — opcionais; se existirem, substituem as
  listas de alimentos embutidas

**Sempre atualize `dias[].atualizado_em` para um horário mais novo** ao editar um
dia — é isso que faz sua edição vencer a versão que está no aparelho.

Subtotais, totais, diferenças e a média de 7 dias são **recalculados pelo app** —
não precisa acertar na mão.

## Privacidade

Repositório público = `dados.json` legível por qualquer um que ache a URL.
Se preferir, mude para **Gist secreto** na configuração do app: o app continua
hospedado no Pages, mas os dados passam a viver num gist não listado (o token
precisa então da permissão `gist`).
