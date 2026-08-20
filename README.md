# Discord-like

Aplicação privada de comunicação para uma única comunidade, inspirada no Discord.

O objetivo é criar uma solução simples para uso pessoal com amigos, priorizando:

- conversa por texto;
- conversa por voz;
- compartilhamento de tela;
- compartilhamento de áudio do sistema quando disponibilizado pelo navegador.

> O nome do projeto é provisório.

## Status

Projeto em fase de planejamento. A primeira implementação será a `v0.1`, focada em validar voz e compartilhamento de tela em duas máquinas Windows usando Chrome e Edge.

## Objetivo inicial

Colocar uma versão funcional nas mãos de 5 a 10 amigos, sem tentar reproduzir todos os recursos do Discord.

O produto final será:

- uma única comunidade;
- acessível apenas por convite;
- executado inicialmente como aplicação web;
- compatível inicialmente com Windows;
- testado inicialmente no Chrome e no Edge;
- projetado para aproximadamente 5–10 usuários.

A `v0.1` será um protótipo técnico local. Ela utilizará tokens temporários de desenvolvimento e não terá autenticação ou sistema real de convites.

## Arquitetura planejada

```text
Cliente React/Vite
        │
        ├── HTTPS ─────── Rust/Axum
        │                  ├── autenticação
        │                  ├── convites
        │                  ├── mensagens
        │                  └── tokens LiveKit
        │
        └── LiveKit SDK ── LiveKit Server
                              ├── voz
                              ├── microfone
                              └── compartilhamento de tela
```

O cliente usa o SDK do LiveKit, que abstrai a comunicação de mídia baseada em WebRTC. O projeto não implementará um servidor WebRTC ou SFU próprio.

### Responsabilidades

#### Rust/Axum

O backend será a autoridade da aplicação e ficará responsável por:

- usuários;
- autenticação e sessões;
- convites;
- permissões;
- canais;
- mensagens;
- emissão de tokens do LiveKit;
- configuração e regras de acesso.

#### React/Vite

O frontend será responsável por:

- interface da aplicação;
- login e navegação;
- exibição do chat;
- controles de microfone;
- conexão com salas de mídia;
- compartilhamento de tela;
- exibição dos participantes.

#### LiveKit Server

O LiveKit Server será responsável pela camada de mídia:

- transporte de áudio e vídeo;
- salas de voz;
- distribuição de microfone;
- distribuição de tela compartilhada;
- conexão entre participantes;
- reconexão de mídia.

A chave secreta do LiveKit nunca será exposta ao frontend. O cliente receberá apenas tokens gerados pelo backend Rust.

## Stack e ferramentas

- Backend: Rust com Axum.
- Frontend: TypeScript com React e Vite.
- Mídia: LiveKit SDK e LiveKit Server.
- Banco planejado: SQLite.
- Documentação: Markdown.
- Cliente desktop futuro: Tauri.

As versões mínimas do Rust, Node.js, npm e demais ferramentas serão registradas em arquivos de configuração do projeto, como `rust-toolchain.toml`, `.nvmrc` ou `package.json`. Os arquivos de lock também deverão ser mantidos no controle de versão.

## Escopo da `v0.1`

A `v0.1` será um vertical slice técnico para validar a parte mais arriscada do projeto.

### Incluído

- React + Vite;
- Rust + Axum;
- endpoint `/health`;
- endpoint temporário para gerar token LiveKit;
- uma sala fixa chamada `geral`;
- conexão de múltiplos participantes;
- microfone;
- ativação e desativação do microfone;
- áudio dos participantes;
- compartilhamento de tela;
- áudio do sistema quando Chrome ou Edge disponibilizar essa faixa;
- indicador de participantes;
- limite de uma transmissão de tela simultânea;
- chat apenas mockado.

### Fora do escopo

- login real;
- convites reais;
- SQLite;
- histórico de mensagens;
- WebSocket próprio;
- múltiplos canais;
- permissões completas;
- câmera;
- aplicativo Tauri;
- aplicativo mobile;
- deploy em produção.

## Configuração da `v0.1`

A primeira versão utilizará uma configuração mínima por variáveis de ambiente:

```text
SERVER_HOST=127.0.0.1
SERVER_PORT=3000

LIVEKIT_URL=ws://127.0.0.1:7880
LIVEKIT_API_KEY=dev_key
LIVEKIT_API_SECRET=dev_secret

WEB_ORIGIN=http://localhost:5173
DEV_MODE=true
```

O endpoint temporário de token somente poderá ser usado em modo de desenvolvimento. O `LIVEKIT_API_SECRET` ficará exclusivamente no backend Rust.

O frontend receberá somente dados como:

```json
{
  "token": "...",
  "url": "ws://127.0.0.1:7880"
}
```

Em desenvolvimento local, `ws://` pode ser utilizado quando o cliente e o LiveKit estiverem acessíveis localmente. Em produção, a URL deverá usar `wss://`, por exemplo:

```json
{
  "token": "...",
  "url": "wss://livekit.seudominio.com"
}
```

O desenvolvimento da `v0.1` será local. Testes entre dois computadores poderão exigir HTTPS local e `wss://`, pois o compartilhamento de tela depende de um contexto seguro.

## Endpoints iniciais

```text
GET  /health
POST /api/livekit/token
```

Na `v0.2`, o endpoint de token passará a exigir uma sessão autenticada e uma autorização válida.

## Critérios de aceite da `v0.1`

A versão será considerada validada quando:

- o backend iniciar com configuração externa;
- `/health` retornar sucesso;
- o frontend solicitar um token ao backend Rust;
- Chrome e Edge conseguirem entrar na mesma sala;
- dois participantes conseguirem conversar por voz;
- o microfone puder ser ativado e desativado;
- uma tela puder ser compartilhada;
- o vídeo da tela compartilhada for recebido;
- o áudio for recebido quando a fonte escolhida disponibilizar áudio;
- somente uma transmissão de tela simultânea for permitida;
- um terceiro participante conseguir entrar;
- sair e entrar novamente funcionar;
- uma interrupção curta de rede permitir reconexão.

O áudio do sistema é um requisito condicionado à fonte escolhida no diálogo de compartilhamento do Chrome ou Edge. Algumas janelas ou fontes podem não disponibilizar uma faixa de áudio.

## Testes da `v0.1`

A validação será dividida em testes explícitos:

```text
M01 — Dois participantes
M02 — Microfone
M03 — Screen share
M04 — Screen share + áudio
M05 — Terceiro participante
M06 — Reentrada
M07 — Reconexão após interrupção
```

### Definição dos testes

- **M01 — Dois participantes:** dois computadores Windows diferentes, um usando Chrome e outro Edge, entram na mesma sala e aparecem na lista.
- **M02 — Microfone:** um participante fala, o outro ouve e o mute interrompe o áudio.
- **M03 — Screen share:** o vídeo da tela é recebido pelo segundo participante.
- **M04 — Screen share + áudio:** o áudio é recebido quando a fonte escolhida disponibiliza uma faixa.
- **M05 — Terceiro participante:** o terceiro participante recebe voz e tela compartilhada.
- **M06 — Reentrada:** o participante sai e consegue entrar novamente.
- **M07 — Reconexão:** após uma interrupção curta da conexão de mídia, o cliente recupera a conexão com o LiveKit.

O teste M04 deve registrar a fonte utilizada, como tela inteira, janela ou aba do navegador, para diferenciar limitações do navegador de problemas da aplicação.

## Roadmap

```text
Fase 0  Especificação e planejamento
   ↓
v0.1    Vertical slice de mídia
        Voz + tela + áudio
   ↓
v0.2    Identidade e acesso privado
        Usuários + login + sessões + convites
   ↓
v0.3    Chat persistente
        SQLite + mensagens + WebSocket
   ↓
v0.4    Produto integrado
        Canais + voz + chat + interface
   ↓
v0.5    Deploy privado
        VPS inicialmente + HTTPS + backups
        Servidor doméstico como alternativa futura
   ↓
v0.6    Estabilidade
        Reconexão + recuperação + testes
   ↓
v1.0    Primeira versão estável
```

### Depois da `v1.0`

- cliente desktop com Tauri;
- notificações desktop;
- câmera;
- reações;
- mensagens fixadas;
- busca;
- anexos;
- emojis personalizados;
- múltiplas transmissões simultâneas.

## Decisões atuais

- O LiveKit será usado desde a `v0.1`.
- A aplicação web virá antes do cliente Tauri.
- Rust será usado no backend com Axum.
- SQLite será usado quando o chat persistente for implementado.
- A primeira versão terá uma única comunidade.
- A primeira versão será focada em Windows, Chrome e Edge.
- O limite inicial será de 5 a 10 usuários.
- O desenvolvimento da `v0.1` será local; produção será planejada posteriormente.
- A VPS será o destino inicial de produção; servidor doméstico será uma alternativa futura.
- Não será implementado um servidor WebRTC ou SFU próprio.
- A tela compartilhada terá inicialmente limite de uma transmissão simultânea.

## Estrutura planejada

```text
discord-like/
├── server/
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs
│       ├── config.rs
│       ├── state.rs
│       ├── error.rs
│       └── routes.rs
│
├── web/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── src/
│
├── infra/
├── docs/
│   ├── roadmap.md
│   ├── v0.1.md
│   ├── architecture.md
│   ├── testing.md
│   └── decisions/
├── .env.example
├── .gitignore
├── rust-toolchain.toml
├── .nvmrc
└── README.md
```

## Documentação planejada

A documentação detalhada será organizada em `docs/` e deverá incluir:

- especificação da `v0.1`;
- arquitetura;
- integração com o LiveKit;
- plano de testes;
- decisões arquiteturais;
- modelo de dados;
- API HTTP;
- eventos em tempo real;
- autenticação e autorização;
- deploy;
- backups e restauração;
- solução de problemas.

## Licença

Ainda não definida.
