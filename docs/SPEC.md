# Especificação resumida (sigclick)

Baseada na skill `linux-autoclicker`. Fonte da verdade: skill salva em `~/NADOLNY_Vault/skills/sigclick.md`.

## Comportamento
- Inicia em *standby* (sem cliques)
- Start key inicia cliques no intervalo configurado
- Stop key para cliques e volta ao standby
- Idempotente; ignora auto-repeat e key release
- Roda até SIGINT/SIGTERM

## Arquitetura
- Ler `/dev/input/event*` com `read()` em `struct input_event` (filtrar EV_KEY)
- Criar mouse virtual via `/dev/uinput` com `ioctl`
- Clique = BTN_DOWN -> SYN_REPORT -> BTN_UP -> SYN_REPORT
- Single thread + `poll()`; timeout = tempo até próximo deadline
- Timing com `CLOCK_MONOTONIC`, deadline = deadline_anterior + intervalo (sem drift)
- Estado: idle/clicking + next_deadline

## Regras duras
- Nunca `EVIOCGRAB`
- Nunca logar teclas
- Checar retorno de todas syscalls
- Tratar `EINTR`
- Cleanup completo em toda saída (UI_DEV_DESTROY, fechar fds, botão não preso)
- Sem threads, sem libevdev, sem XTest/xdotool/system
- C11, `-Wall -Wextra -Wpedantic`, sem warnings

## Configuração
Ver README.md
