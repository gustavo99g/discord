# Roadmap do projeto

Este documento define a evolução planejada do projeto, desde a validação técnica inicial até uma versão estável para uso privado entre amigos.

## Objetivo do projeto

Criar uma aplicação privada de comunicação para uma única comunidade, com foco em:

- chat por texto;
- conversa por voz;
- compartilhamento de tela;
- compartilhamento de áudio do sistema quando disponibilizado pelo navegador.

## Premissas atuais

- Sistema operacional inicial: Windows.
- Navegadores iniciais: Chrome e Edge.
- Usuários simultâneos esperados: 5–10.
- Plataforma inicial: aplicação web.
- Comunidade: uma única.
- Mídia: LiveKit desde a primeira versão.
- Backend: Rust com Axum.
- Banco planejado: SQLite.
- Cliente desktop com Tauri: somente depois da primeira versão estável.
- Desenvolvimento da `v0.1`: local, sem exigência de produção.
- Produção inicial: VPS; servidor doméstico como alternativa futura.

---

# Fase 0 — Especificação e planejamento

## Objetivo

Definir o comportamento da primeira versão antes da implementação.

## Entregas

- Documento de arquitetura.
- Especificação técnica da `v0.1`.
- Fluxo de conexão com o LiveKit.
- Critérios de aceite.
- Matriz de testes para Windows, Chrome e Edge.
- Registro das decisões arquiteturais.
- Lista de riscos conhecidos.

## Critério de conclusão

A equipe consegue explicar como executar, testar e validar a `v0.1` sem precisar tomar decisões arquiteturais fundamentais durante a implementação.

---

# `v0.1` — Vertical slice de mídia

## Objetivo

Validar a experiência principal do produto e o maior risco técnico: voz e compartilhamento de tela com áudio.

## Funcionalidades

- React + Vite.
- Rust + Axum.
- Endpoint `GET /health`.
- Endpoint temporário para geração de token LiveKit.
- Sala fixa chamada `geral`.
- Conexão de múltiplos participantes.
- Microfone.
- Ativação e desativação do microfone.
- Áudio dos participantes.
- Compartilhamento de tela.
- Áudio do sistema quando Chrome ou Edge disponibilizar essa faixa.
- Indicador de participantes.
- Limite de uma transmissão de tela simultânea.
- Chat apenas mockado.
- Reconexão básica da conexão de mídia fornecida pelo LiveKit.

## Fora do escopo

- Login real.
- Convites.
- SQLite.
- Histórico de mensagens.
- WebSocket próprio.
- Múltiplos canais.
- Permissões completas.
- Câmera.
- Tauri.
- Deploy em produção.

## Critérios de aceite

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
- uma interrupção curta da conexão de mídia permitir reconexão básica pelo LiveKit.

## Observação sobre o áudio da tela

O áudio do sistema é condicionado à fonte escolhida no diálogo de compartilhamento do Chrome ou Edge. Algumas janelas ou fontes podem não disponibilizar uma faixa de áudio.

O requisito inicial é:

```text
Vídeo da tela: obrigatório
Áudio da tela: obrigatório quando disponibilizado pelo navegador
```

## Resultado esperado

Duas máquinas Windows, uma usando Chrome e outra Edge, conseguem entrar na sala, conversar por voz e compartilhar uma tela com áudio quando a fonte permitir.

A `v0.1` é uma exceção de desenvolvimento local: utiliza tokens temporários e não possui autenticação ou convites reais.

---

# `v0.2` — Identidade e acesso privado

## Objetivo

Substituir o acesso temporário por autenticação e controle de entrada.

## Funcionalidades

- SQLite.
- Usuários.
- Cadastro por convite.
- Login.
- Logout.
- Sessões.
- Expiração de sessão.
- Papéis `owner` e `member`.
- Convites com validade.
- Limite de usos por convite.
- Revogação de convites.
- Tokens LiveKit emitidos somente para usuários autorizados.

## Fluxo principal

```text
Convite
  ↓
Cadastro
  ↓
Login
  ↓
Sessão
  ↓
Rust valida o usuário
  ↓
Rust verifica a permissão
  ↓
Rust gera token LiveKit
  ↓
Cliente entra na sala
```

## Critério de conclusão

Uma pessoa sem convite não consegue criar uma conta ou entrar na sala de mídia.

A restrição por convite aplica-se ao produto privado a partir da `v0.2`. A `v0.1` é um protótipo local e utiliza tokens temporários de desenvolvimento.

---

# `v0.3` — Chat persistente

## Objetivo

Adicionar comunicação por texto com histórico e atualização em tempo real.

## Funcionalidades

- Canal `#geral`.
- SQLite para mensagens.
- Envio de mensagens.
- Histórico de mensagens.
- Paginação.
- WebSocket autenticado.
- Mensagens em tempo real.
- Reconexão básica.
- Limite de tamanho das mensagens.

## Entidades iniciais

```text
users
sessions
invites
channels
messages
```

## Eventos iniciais

```text
message.created
message.updated
message.deleted
```

## Critério de conclusão

Dois usuários conseguem trocar mensagens em tempo real e continuam vendo o histórico após reiniciar o servidor.

---

# `v0.4` — Produto integrado

## Objetivo

Unificar chat, voz e compartilhamento de tela em uma interface coerente.

## Funcionalidades

- Lista de canais.
- Canal de texto.
- Canal de voz.
- Entrada e saída da sala.
- Lista de participantes.
- Estado do microfone.
- Compartilhamento de tela.
- Chat real.
- Presença online/offline.
- Evento `presence.updated`.
- Interface visual organizada.
- Estados de carregamento.
- Mensagens de erro.

## Critério de conclusão

Um usuário consegue entrar, conversar por texto, entrar na sala de voz e compartilhar a tela sem usar ferramentas de desenvolvimento.

---

# `v0.5` — Deploy privado

## Objetivo

Disponibilizar o sistema para os amigos por meio de uma instalação acessível remotamente.

## Infraestrutura planejada

```text
Caddy
  ├── Frontend web
  ├── Rust/Axum
  └── LiveKit Server
```

## Funcionalidades e tarefas

- Deploy inicialmente em VPS.
- Servidor doméstico como alternativa futura.
- HTTPS.
- WebSocket seguro.
- LiveKit self-hosted.
- Configuração de conectividade WebRTC.
- Variáveis de ambiente.
- Backups do SQLite.
- Logs.
- Documentação de instalação.
- Configuração de firewall.
- Rotina de atualização.

## Critério de conclusão

Os usuários conseguem acessar a aplicação por um endereço remoto, sem depender do ambiente de desenvolvimento.

---

# `v0.6` — Estabilidade e operação

## Objetivo

Tornar o sistema confiável para uso recorrente.

## Funcionalidades

- Reconexão automática do WebSocket.
- Reconexão robusta das salas de mídia.
- Tratamento de queda de internet.
- Expiração e revogação de sessões.
- Revogação de convites.
- Limites de requisições.
- Proteção contra mensagens muito grandes.
- Logs estruturados.
- Testes de recuperação.
- Validação de backup e restauração.
- Testes com 5–10 usuários.
- Tratamento de falhas do LiveKit.

## Critério de conclusão

O sistema pode ser usado regularmente sem exigir intervenção manual após falhas comuns de rede ou reinicializações.

---

# `v1.0` — Primeira versão estável

## Objetivo

Entregar uma versão confiável para uso cotidiano entre os amigos.

## Requisitos

- Cadastro por convite.
- Login e sessões.
- Chat persistente.
- Chat em tempo real.
- Voz estável.
- Microfone.
- Compartilhamento de tela.
- Áudio da tela quando suportado pela fonte do navegador.
- Permissões básicas.
- HTTPS.
- Backups.
- Recuperação do sistema.
- Suporte para 5–10 usuários.
- Compatibilidade com Chrome e Edge no Windows.

## Fluxo completo

```text
Receber convite
  ↓
Criar conta
  ↓
Fazer login
  ↓
Conversar por texto
  ↓
Entrar na sala de voz
  ↓
Usar o microfone
  ↓
Compartilhar a tela com áudio
  ↓
Usar o sistema com segurança
```

A `v1.0` prioriza estabilidade sobre quantidade de funcionalidades.

---

# Versões posteriores

## `v1.1` — Cliente desktop

- Tauri.
- Instalador para Windows.
- Atalhos de teclado.
- Notificações desktop.
- Inicialização automática opcional.
- Reutilização do frontend React existente.

## `v1.2` — Recursos sociais

- Reações.
- Mensagens fixadas.
- Respostas.
- Busca.
- Anexos.
- Emojis personalizados.
- Status personalizados.

## `v1.3` — Recursos avançados de mídia

- Câmera.
- Múltiplas transmissões simultâneas.
- Seleção de qualidade.
- Controle individual de volume.
- Push-to-talk.
- Indicador de quem está falando.
- Áudio de fontes adicionais.

---

# Fora do escopo inicial

Os seguintes itens não devem ser implementados antes da `v1.0`, salvo mudança explícita de prioridade:

- aplicativo mobile;
- múltiplas comunidades;
- bots;
- descoberta pública;
- federação;
- gravação de chamadas;
- criptografia ponta a ponta própria;
- marketplace;
- sistema complexo de moderação;
- streaming público;
- sistema completo de câmera e vídeo.

---

# Resumo visual

```text
Fase 0  Especificação
   ↓
v0.1    Voz + tela + áudio funcionando
   ↓
v0.2    Login + sessões + convites
   ↓
v0.3    SQLite + mensagens + WebSocket
   ↓
v0.4    Produto integrado
   ↓
v0.5    Deploy privado
   ↓
v0.6    Estabilidade
   ↓
v1.0    Uso cotidiano pelos amigos
   ↓
v1.1    Cliente Tauri
```
