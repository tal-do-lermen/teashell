# Roadmap de Pesquisa — Desktop Shell Wayland em Rust

Este documento é um roteiro do que pesquisar para implementar um **desktop shell para Wayland em Rust**, baseado no guia fornecido. O foco é entender os conceitos e tecnologias antes de implementar.

> Objetivo: construir um shell modular com barra, launcher, notificações, lock screen, dock, wallpaper, control center, widgets, configuração e IPC, executando sobre um compositor Wayland existente como Sway, Hyprland ou Niri.

---

## 1. Rust para aplicações de sistema

Pesquise:

- Ownership e Borrowing
- Lifetimes
- `struct`, `enum` e traits
- `Result`, `Option` e tratamento de erros
- `thiserror` e `anyhow`
- `Arc`, `Mutex`, `RwLock`
- Channels (`std::sync::mpsc` e canais assíncronos)
- Threads
- Async/Await
- Tokio
- Event loops
- Filesystem
- Processos e sinais
- Unix sockets
- Logging
- `tracing` e `tracing-subscriber`

Objetivo: conseguir construir uma aplicação de longa duração, orientada a eventos e com vários subsistemas independentes.

---

# 2. Wayland — fundamentos

Antes de usar bibliotecas de alto nível, entenda o protocolo.

Pesquise:

- Arquitetura do Wayland
- Client vs Compositor
- Wayland Protocol
- Globals
- Registry
- Object lifecycle
- Proxy objects
- Requests
- Events
- Event queue
- Roundtrip
- `wl_display`
- `wl_registry`
- `wl_compositor`
- `wl_surface`
- `wl_shm`
- `wl_seat`
- `wl_output`
- Frame callbacks
- Configure events
- Commit

Fluxo conceitual que você precisa entender:

```text
conectar ao compositor
        ↓
obter registry
        ↓
descobrir globals
        ↓
obter interfaces necessárias
        ↓
criar surface
        ↓
receber configure
        ↓
renderizar
        ↓
commit
        ↓
aguardar frame callback
        ↓
renderizar novamente quando necessário
```

---

# 3. Rust + Wayland

Pesquise principalmente:

- `wayland-client`
- `wayland-backend`
- `wayland-protocols`
- geração de bindings de protocolos
- Dispatch
- EventQueue
- Registry
- Global manager
- Proxy objects

Você precisa entender como uma API Wayland em Rust transforma:

```text
protocolo Wayland
        ↓
XML do protocolo
        ↓
bindings Rust
        ↓
interfaces
        ↓
requests/events
```

Não comece pelo Smithay. Para um desktop shell executando sobre Sway/Hyprland/Niri, você está construindo principalmente um **cliente Wayland**, não um compositor.

---

# 4. Layer Shell

Esta é uma das partes mais importantes para o shell.

Pesquise:

- `wlr-layer-shell`
- `zwlr_layer_shell_v1`
- `zwlr_layer_surface_v1`
- Layer surfaces
- Background layer
- Bottom layer
- Top layer
- Overlay layer
- Anchors
- Margins
- Exclusive zone
- Keyboard interactivity
- Output selection
- Namespace

Entenda como uma barra pode informar ao compositor:

```text
"quero ficar no topo"
"quero ocupar esta área"
"quero reservar espaço para outros aplicativos"
```

Pesquise também as diferenças entre:

- janela normal
- layer surface
- popup
- subsurface

---

# 5. Rendering

Pesquise:

- GPU rendering
- Vulkan
- EGL
- OpenGL ES
- Wayland EGL
- DMA-BUF
- GPU buffers
- Texture atlas
- GPU compositing
- VSync
- Frame callbacks
- Frame pacing
- Damage tracking
- Partial rendering
- Dirty rectangles

Bibliotecas/tecnologias para estudar:

- `wgpu`
- Skia
- Vello

Para texto:

- `glyphon`
- `cosmic-text`

Compare as abordagens antes de escolher uma.

---

# 6. UI Toolkit

Antes de escolher uma biblioteca, entenda:

- Widget tree
- Layout engine
- Retained mode
- Immediate mode
- Hit testing
- Focus management
- Keyboard navigation
- Mouse/pointer events
- State management
- Reactive UI
- Rendering pipeline

Pesquise:

- Iced
- Slint
- egui
- Cosmic Toolkit

Para aprender a arquitetura, vale implementar primeiro uma pequena camada própria de UI, mesmo que posteriormente você adote um toolkit.

---

# 7. Input

Pesquise:

- `wl_seat`
- `wl_pointer`
- `wl_keyboard`
- Keyboard focus
- Pointer focus
- Keyboard modifiers
- Key repeat
- Cursor
- XCursor
- Pointer motion
- Button events
- Keyboard key events

Entenda como o shell recebe eventos e como eles chegam aos componentes:

```text
Wayland
  ↓
input
  ↓
event dispatcher
  ↓
componente
  ↓
ação
```

---

# 8. Arquitetura interna

Estude uma arquitetura:

```text
Wayland Event Loop
        ↓
Application State
        ↓
Message/Event Bus
        ↓
Modules
```

Separe conceitualmente:

- Bar
- Launcher
- Notifications
- Dock
- Control Center
- Lock Screen
- Wallpaper
- Widgets
- Configuration
- D-Bus services
- IPC

Pesquise:

- Event-driven architecture
- State machines
- Message passing
- Actor model
- Observer pattern
- Dependency inversion
- Modular architecture

Evite criar um grande estado global compartilhado por todos os módulos.

---

# 9. D-Bus

Pesquise:

- D-Bus
- Session Bus
- System Bus
- Object
- Interface
- Method
- Property
- Signal
- ObjectManager
- D-Bus introspection

Biblioteca Rust:

- `zbus`

Entenda principalmente a diferença entre:

```text
Method
Property
Signal
```

e quando utilizar cada um.

---

# 10. Notificações

Pesquise:

- `org.freedesktop.Notifications`
- Desktop Notifications Specification
- `Notify`
- `GetCapabilities`
- `GetServerInformation`
- Actions
- Hints
- Timeout
- Notification replacement
- Notification IDs

Arquitetura:

```text
Aplicação
   ↓
D-Bus
   ↓
Notification service
   ↓
Shell
   ↓
Notification UI
```

---

# 11. Áudio

Em sistemas Linux modernos, pesquise primeiro:

- PipeWire
- WirePlumber
- PipeWire graph
- Nodes
- Devices
- Sinks
- Sources
- Default sink
- Default source
- Volume
- Mute
- Metadata

Também estude:

- PulseAudio compatibility layer

Para o shell, a arquitetura deve considerar PipeWire/WirePlumber em vez de depender exclusivamente da API antiga do PulseAudio.

---

# 12. NetworkManager

Pesquise:

- NetworkManager D-Bus API
- `org.freedesktop.NetworkManager`
- Devices
- Access Points
- Connections
- Profiles
- Connection state
- Wi-Fi scanning
- Authentication
- Secrets

Evite usar parsing de `nmcli` como arquitetura principal.

Pode ser útil para debugging, mas o shell deve preferencialmente conversar com o serviço através de uma API estruturada.

---

# 13. Bluetooth

Pesquise:

- BlueZ
- BlueZ D-Bus API
- ObjectManager
- `Adapter1`
- `Device1`
- `Agent1`
- Discovery
- Pairing
- Connecting
- Disconnecting
- Device properties

Entenda como o shell pode acompanhar mudanças através de D-Bus signals.

---

# 14. Energia

Pesquise:

- UPower
- UPower D-Bus API
- Battery
- Charging state
- Percentage
- Power profiles
- Power Profiles Daemon

Entenda a diferença entre:

- informação da bateria
- gerenciamento de energia
- perfil de performance

---

# 15. Brilho

Para displays internos:

- Linux backlight
- `/sys/class/backlight`
- brightness interfaces
- `brightnessctl`

Para monitores externos:

- DDC/CI
- MCCS
- VCP
- `ddcutil`

Pesquise também:

- Display identification
- EDID
- Monitor capabilities

---

# 16. Launcher

Pesquise:

- Desktop Entry Specification
- `.desktop` files
- XDG Base Directory Specification
- `/usr/share/applications`
- `~/.local/share/applications`
- `Exec`
- `TryExec`
- `Icon`
- `Categories`
- `NoDisplay`
- `Hidden`
- `Terminal`

Funcionalidades:

- indexação
- busca fuzzy
- favoritos
- recentes
- categorias
- autocomplete
- execução de aplicativos

Também pesquise parsing seguro de argumentos do campo `Exec`.

---

# 17. Workspaces e janelas

Pesquise os protocolos:

- `wlr-foreign-toplevel-management`
- `ext-workspace-v1`
- `ext-foreign-toplevel-list-v1`

Entenda:

- listar janelas
- detectar janela ativa
- minimizar/maximizar quando suportado
- fechar
- focar
- workspace atual
- mover janela
- acompanhar eventos

Também pesquise APIs específicas de:

- Sway IPC
- Hyprland IPC
- Niri IPC

Mantenha uma camada de abstração para que o shell não fique preso a um único compositor.

---

# 18. Lock Screen

Pesquise:

- `ext-session-lock-v1`
- Session lock
- Lock surface
- Keyboard exclusive access
- Unlock flow
- PAM
- Linux-PAM
- autenticação em Rust

Importante:

**Session lock não deve ser tratado simplesmente como uma janela layer-shell comum.**

Estude também:

- race conditions
- secure input
- foco exclusivo
- encerramento seguro
- tratamento de erros de autenticação

---

# 19. Idle Detection

Pesquise:

- `ext-idle-notify-v1`
- `org_kde_kwin_idle`
- Idle timeout
- Idle inhibitor
- Screen lock
- DPMS

Exemplo conceitual:

```text
sem interação
      ↓
idle timeout
      ↓
ação
      ↓
lock / display power
```

---

# 20. Wallpaper

Pesquise:

- Layer-shell background
- `wl_output`
- Multi-monitor
- Image decoding
- Texture upload
- GPU textures
- DMA-BUF
- Scaling
- Fractional scaling
- Aspect ratio
- Crop
- Fit
- Fill

Rust:

- crate `image`

Planeje desde o início:

- múltiplos monitores
- wallpapers diferentes
- mudança dinâmica
- cache
- baixo uso de memória
- atualização sem travar a UI

---

# 21. Multi-monitor

Pesquise:

- `wl_output`
- Output lifecycle
- Output scale
- Logical coordinates
- Physical dimensions
- Refresh rate
- Fractional scaling
- Output enter/leave

O shell deve possuir um estado por monitor:

```text
Output
 ├── Bar
 ├── Wallpaper
 ├── Widgets
 └── UI state
```

---

# 22. Configuração

Pesquise:

- Serde
- TOML
- XDG Base Directory
- XDG config directory
- Config validation
- Config reload
- File watching

Rust:

- `serde`
- `toml`
- `notify`

Estrutura conceitual:

```text
config.toml
      ↓
parser
      ↓
validation
      ↓
application state
```

Para hot reload:

```text
arquivo alterado
      ↓
watcher
      ↓
parse
      ↓
validate
      ↓
apply
```

---

# 23. IPC

Pesquise:

- Unix Domain Socket
- `UnixListener`
- `UnixStream`
- Socket permissions
- IPC protocol design
- JSON-RPC
- Length-prefixed protocol
- Request/Response
- Events

Arquitetura:

```text
CLI
 ↓
Unix socket
 ↓
Shell
 ↓
Command dispatcher
 ↓
State
```

Comandos possíveis:

- reload
- toggle launcher
- toggle control center
- show notification
- change wallpaper
- lock
- quit
- query state

---

# 24. Performance

Como o shell ficará sempre executando, performance deve ser estudada desde o começo.

Pesquise:

- Linux `perf`
- `cargo flamegraph`
- Flamegraphs
- Heap profiling
- Allocation profiling
- GPU profiling
- RenderDoc
- Sysprof
- `strace`
- `bpftrace`
- eBPF
- `criterion`
- `tracing`

Para Wayland:

- Frame callbacks
- Damage tracking
- Partial redraw
- Dirty regions
- Frame pacing
- VSync
- Input latency
- GPU synchronization

Evite uma arquitetura baseada em:

```text
loop
  ↓
sleep(16ms)
  ↓
redraw everything
```

Prefira:

```text
event
 ↓
state changed?
 ↓
damage
 ↓
render
 ↓
frame callback
 ↓
wait
```

---

# 25. Estrutura conceitual do projeto

Uma possível divisão:

```text
Application
├── Wayland
├── Renderer
├── Input
├── State
├── Config
├── IPC
├── D-Bus
│   ├── Notifications
│   ├── Audio
│   ├── Network
│   ├── Bluetooth
│   └── Power
│
├── UI
│   ├── Bar
│   ├── Launcher
│   ├── Dock
│   ├── Control Center
│   ├── Notifications
│   ├── Lock Screen
│   └── Widgets
│
└── Wallpaper
```

A ideia principal é evitar que os componentes de UI conheçam diretamente detalhes de D-Bus ou Wayland.

---

# 26. Ordem recomendada de implementação

## Fase 1 — Rust

Estude:

- ownership
- concurrency
- async
- channels
- errors
- filesystem
- processes
- logging

## Fase 2 — Wayland Client

Estude:

- `wayland-client`
- registry
- globals
- event queue
- surface
- configure
- commit
- frame callback

## Fase 3 — Layer Shell

Implemente conceitualmente:

- surface top
- anchors
- exclusive zone
- output
- keyboard interaction

## Fase 4 — Rendering

Estude e escolha:

- wgpu
- Vulkan
- OpenGL ES
- Skia
- Vello

Depois:

- text
- images
- textures
- GPU buffers
- damage tracking

## Fase 5 — Input

Estude:

- pointer
- keyboard
- focus
- modifiers
- repeat
- cursor

## Fase 6 — Bar

Comece com:

- relógio
- workspaces
- status
- launcher button

Depois adicione:

- áudio
- rede
- Bluetooth
- bateria
- indicadores

## Fase 7 — D-Bus

Implemente serviços separadamente:

1. Notifications
2. Audio
3. NetworkManager
4. Bluetooth
5. UPower

## Fase 8 — Launcher

Pesquise:

- `.desktop`
- XDG
- indexação
- fuzzy search
- processo de execução

## Fase 9 — Control Center

Integre:

- áudio
- rede
- Bluetooth
- brilho
- bateria
- energia

## Fase 10 — Windows/Workspaces

Integre:

- foreign toplevel
- workspaces
- compositor-specific IPC

## Fase 11 — Lock

Pesquise:

- session lock
- PAM
- secure input
- idle

## Fase 12 — Wallpaper

Adicione:

- múltiplos monitores
- scaling
- cache
- GPU textures

## Fase 13 — Config + IPC

Finalize:

- TOML
- hot reload
- Unix socket
- CLI

## Fase 14 — Otimização

Só depois de funcional:

- profiling
- allocations
- GPU profiling
- redraw optimization
- startup time
- memory usage
- input latency

---

# 27. O que NÃO estudar primeiro

Não comece por:

- Smithay internals
- DRM/KMS
- compositor implementation
- XWayland internals
- wlroots internals
- GPU driver development

Esses assuntos são muito importantes para **um compositor Wayland**, mas não são necessários para o primeiro estágio de um desktop shell.

Se posteriormente o objetivo for criar seu próprio compositor/WM em Rust, aí sim eles passam a ser centrais.

---

# 28. Recursos para pesquisa

Pesquise sempre primeiro nas documentações oficiais/projetos:

- Wayland Documentation
- Wayland Protocols
- `wayland-client`
- `wayland-backend`
- `wayland-protocols`
- `wlr-layer-shell`
- Smithay
- wlroots
- wgpu
- PipeWire
- WirePlumber
- D-Bus
- zbus
- NetworkManager
- BlueZ
- UPower
- Desktop Entry Specification
- XDG Specifications
- Sway IPC
- Hyprland IPC
- Niri IPC
- Noctalia
- Iced
- Slint
- egui
- Cosmic Toolkit

---

# 29. Conceito geral

A arquitetura final deve se aproximar de:

```text
                   WAYLAND COMPOSITOR
                 Sway / Hyprland / Niri
                          │
                          │ Wayland
                          ▼
                 ┌──────────────────┐
                 │   Desktop Shell  │
                 │                  │
                 │ ┌──────────────┐ │
                 │ │ Event Loop   │ │
                 │ └──────┬───────┘ │
                 │        │         │
                 │ ┌──────▼───────┐ │
                 │ │ Application  │ │
                 │ │    State     │ │
                 │ └──────┬───────┘ │
                 │        │         │
                 │ ┌──────▼────────┐│
                 │ │ UI / Modules  ││
                 │ │               ││
                 │ │ Bar           ││
                 │ │ Launcher      ││
                 │ │ Dock          ││
                 │ │ Notifications ││
                 │ │ Control Center││
                 │ │ Lock Screen   ││
                 │ │ Wallpaper     ││
                 │ └───────────────┘│
                 └──────────────────┘
                     │           │
                  Wayland       D-Bus
                     │           │
                     │     ┌─────┴──────────┐
                     │     │                │
                     │  PipeWire      NetworkManager
                     │  BlueZ         UPower
                     │  etc.          etc.
                     │
                     ▼
                   GPU
```

O objetivo da pesquisa é entender cada camada individualmente antes de conectá-las.

---

# 30. Resultado esperado

Ao terminar a pesquisa, você deve conseguir explicar sem consultar código:

1. Como um cliente Wayland se conecta ao compositor.
2. Como descobre globals.
3. Como cria uma surface.
4. Como transforma uma surface em layer-shell.
5. Como recebe input.
6. Como renderiza.
7. Como sincroniza frames.
8. Como detecta mudanças de estado.
9. Como conversa com D-Bus.
10. Como controla áudio/rede/Bluetooth/energia.
11. Como encontra aplicações `.desktop`.
12. Como acompanha janelas/workspaces.
13. Como implementa lock screen.
14. Como monitora idle.
15. Como gerencia múltiplos monitores.
16. Como mantém configuração.
17. Como cria IPC.
18. Como mede e otimiza performance.

Só depois disso vale começar a montar a implementação completa.
