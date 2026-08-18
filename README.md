# Sala de Oração (Sexta) — Piloto de App de Apoio Musical

Piloto de uma aplicação para músicos iniciantes/intermediários da Sala de Oração, com foco em leitura por **grau (Nashville / cifra em graus)**, transposição em tempo real e apoio auditivo (identificação de nota e conferência de acorde).

Testado com a cifra real de **"Tudo É Perda" — Felipe Rodrigues** (tom D, capo 0), extraída do Cifra Club.

---

## Critérios e decisões de projeto

### 1. Público-alvo define a prioridade
O app foi pensado para músicos **não profissionais**, tocando ao vivo, muitas vezes olhando o celular no pé de partitura. Isso guiou três critérios de design:
- **Legibilidade em tela pequena** antes de densidade de informação.
- **Transposição instantânea**, já que o tom de sábado pode mudar por causa da voz de quem canta.
- **Grau como formato principal**, cifra como alternativa — porque é assim que o time já pensa a música (Nashville Number System / cifra em graus, comum em música de adoração brasileira).

### 2. Modelo de dados: grau como estrutura, não como texto solto
Em vez de guardar a cifra como texto livre, cada acorde é um objeto:
```
{ deg, extra?, over? }
```
- `deg` → grau diatônico (0=I, 1=ii, 2=iii, 3=IV, 4=V, 5=vi, 6=vii°)
- `extra` → extensão (ex: `"7"`, `"7M(2)"`)
- `over` → grau do baixo, para acordes invertidos/slash (ex: D/F# = `over: iii`)

**Por quê:** isso permite gerar cifra e grau a partir da mesma fonte, e transpor só trocando o "tom base" — sem reescrever nada manualmente. Foi um critério não-negociável, porque transposição feita "na mão" (reescrevendo cada acorde) é a principal fonte de erro em cifras de igreja.

### 3. Acordes com baixo diferente da fundamental (slash chords) tratados explicitamente
A cifra real trouxe casos como `G7M(2)/D` e `D/F#`, que um conversor simples de "achar e trocar letra" quebraria. A solução foi separar **grau do acorde** e **grau do baixo** como dois campos independentes, cada um transposto pela mesma fórmula.

### 4. Estrutura em seções com repetição anotada, não repetição literal
Em vez de listar o mesmo trecho de acordes várias vezes na tela (o que um músico de igreja já sabe ler como "2x" ou "4x"), cada seção guarda o ciclo de acordes **uma vez** e um contador de repetição (`×2`, `×4`). Critério: reduzir ruído visual sem perder informação musical.

### 5. Detecção de áudio dividida em dois modos, com expectativa realista
Esse foi o ponto mais sensível do projeto — separar o que é **tecnicamente confiável hoje** do que é **pesquisa em andamento**:

| Modo | O que faz | Técnica | Confiabilidade |
|---|---|---|---|
| **Nota isolada** | Identifica uma nota tocada sozinha (afinador) | Autocorrelação (ACF) no domínio do tempo | Alta — funciona bem, é o mesmo princípio de afinadores de instrumento |
| **Banda (beta)** | Confere se o áudio bate com um acorde **já esperado pela cifra** | Vetor de croma (energia por classe de nota) comparado a *templates* de acorde (maj/min/dim) | Assistida, não uma transcrição livre |

**Critério central do modo Banda:** em vez de tentar "adivinhar qualquer acorde do mundo" a partir de um mix completo (bateria + baixo + várias vozes) — problema difícil mesmo em ferramentas comerciais de MIR — o app **restringe a busca aos acordes que já aparecem naquela música**. O músico marca manualmente qual acorde é o esperado (tocando na cifra), e o app só precisa responder "bate ou não bate", que é um problema muito mais tratável do que transcrição aberta.

Limitações assumidas conscientemente e comunicadas na interface:
- Não sincroniza automaticamente com o tempo/compasso — a navegação é manual (‹ anterior / próximo ›).
- Ignora a nota de baixo/inversão na comparação (trata `D/F#` como `D` para fins de detecção).
- Degrada com bateria muito presente ou grave dominante no mix.

### 6. Sem reprodução de letra
Por critério de direitos autorais, o piloto guarda e exibe **apenas a estrutura harmônica** (acordes/graus por seção), nunca o texto da letra — mesmo tendo acesso à cifra completa com letra.

### 7. Identidade visual como critério funcional, não só estético
- Fundo escuro e paleta quente (dourado/âmbar) — referência a vela/oração, mas também **reduz cansaço visual** em ambiente de culto com luz baixa.
- Acordes exibidos como "tags penduradas" com uma linha de pauta acima — metáfora do quadro de hinos físico, para que o layout seja reconhecível para quem já usa esse formato.
- Fontes: serifada (títulos/seções) + monoespaçada (acordes/graus, para alinhamento e leitura técnica rápida).

### 8. Stack e hospedagem
- HTML/CSS/JS puro, sem dependência de build — critério: qualquer pessoa da equipe consegue abrir, editar ou hospedar sem toolchain.
- Web Audio API nativa (sem biblioteca externa) para captura e análise de áudio.
- Hospedagem em GitHub Pages como `index.html` — necessário para acesso ao microfone em celular, que exige HTTPS (não funciona em arquivo local `file://` no navegador mobile).

---

## Limitações conhecidas (em aberto para próxima fase)
- Progressão de acordes é cadastrada manualmente por música (sem integração automática ainda com o Cifra Club).
- Sem BPM/andamento nem sincronização automática de posição na música.
- Detecção de acorde não extraída do banco de dados do Notion — piloto usa dados fixos no código.
- Sem persistência de repertório: cada música precisa ser codificada à mão até haver integração com o Notion.

## Próximos passos possíveis
- Conectar ao banco do Notion (`🎶 Banco de Músicas - Sala de Oração (Sexta)`) para carregar músicas dinamicamente.
- Adicionar campos propostos no Notion: `Artista/Autor`, `Cifra em Grau`, `Tom Original`, `BPM`, `Link Cifra`.
- Testar o Modo Banda com a banda completa tocando e calibrar limiares de confiança com dados reais.
