# sigclick

Um autoclicker Linux educativo, escrito em C, utilizando **evdev** (`/dev/input/event*`) para capturar teclas de atalho e **uinput** (`/dev/uinput`) para simular cliques do mouse.

> Projeto com fins educacionais (aprendizado de syscalls Linux, ioctl, poll, sinais, etc). Código escrito pelo próprio aluno, com mentoria.

## Objetivo

- Rodar em segundo plano em modo *standby* (sem cliques).
- Ao pressionar a tecla de *start* (padrão: `F6`), iniciar cliques automáticos no intervalo configurado.
- Ao pressionar a tecla de *stop* (padrão: `F7`), parar e voltar ao modo *standby*.
- Funcionar no X11 e Wayland (via kernel input).
- Usar apenas libc + headers do kernel (sem libevdev, XTest, xdotool).

## Tecnologias

- C11
- evdev (`struct input_event`, `EV_KEY`)
- uinput (dispositivo virtual de mouse)
- `poll()` single-thread
- `clock_gettime(CLOCK_MONOTONIC)` para timing sem drift
- `sigaction` para parada limpa

## Requisitos do sistema

- Linux (kernel com suporte a evdev e uinput)
- Permissões para acessar `/dev/input/*` e `/dev/uinput`
- Compilador C (gcc/clang)

### Permissões

Recomendado adicionar seu usuário ao grupo `input` e configurar udev para `/dev/uinput`:

```bash
sudo usermod -aG input $USER
# Após relogar
```

Para testes rápidos também é possível usar `sudo`.

## Build

### Com CMake (recomendado)

```bash
mkdir -p build
cd build
cmake ..
make
```

### Com Makefile (se disponível)

```bash
make
```

## Uso

```bash
./sigclick -d /dev/input/by-id/SEU-TECLADO-event-kbd
```

### Opções

| Opção | Longa | Descrição | Padrão |
|---|---|---|---|
| `-i` | `--interval` | Intervalo entre cliques em milissegundos | `100` |
| `-s` | `--start-key` | Tecla para iniciar cliques | `F6` |
| `-x` | `--stop-key` | Tecla para parar cliques | `F7` |
| `-b` | `--button` | Botão: `left`, `right`, `middle` | `left` |
| `-d` | `--device` | Caminho do dispositivo teclado (requerido) | - |
| `-h` | `--help` | Exibe ajuda | - |

Exemplo:
```bash
./sigclick -d /dev/input/by-id/usb-Logitech_G213_Prodigy_Gaming_Keyboard-event-kbd -i 50 -s F8 -x F9 -b left
```

## Como encontrar seu teclado

```bash
ls -la /dev/input/by-id/ | grep kbd
# ou
cat /proc/bus/input/devices | grep -A 5 -B 5 "keyboard\|kbd"
```

Para testar eventos:
```bash
evtest /dev/input/by-id/SEU-TECLADO-event-kbd
```

## Segurança e robustez

- Nunca faz `EVIOCGRAB` (não rouba o teclado do sistema)
- Nunca loga teclas digitadas
- Só reage a `EV_KEY` com `value == 1` (pressionar). Ignora `value == 0` (soltar) e `value == 2` (auto-repeat)
- Usa `clock_gettime(CLOCK_MONOTONIC)`, avança deadline pelo intervalo anterior (sem drift)
- `poll()` com timeout dinâmico até próximo clique
- Tratamento de `EINTR`, checagem de retorno de syscalls
- Na saída (normal/sinal/erro): garante que botão não fique pressionado, `UI_DEV_DESTROY`, fecha fds
- Roda com ~0% CPU em modo standby

## Milestones (desenvolvimento)

1. **Ler teclas** - Programa descartável para imprimir códigos de tecla via evdev
2. **Clique único** - Criar mouse virtual com uinput e emitir 1 clique
3. **Toggle com F6/F7** - Loop com `poll()` + deadline timing
4. **Configuração** - `getopt_long`, tabela de nomes de teclas, validação
5. **Robustez** - Sinais, erros, cleanup completo, limpo com sanitizers/valgrind
6. **Polimento (opcional)** - Auto-detecção, README, etc

## Desenvolvimento

Compilar com warnings estritos:
```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
# ou com sanitizers
cmake -DCMAKE_BUILD_TYPE=Debug -DENABLE_SANITIZERS=ON ..
```

## Licença

Projeto educacional. Uso livre para aprendizado.
