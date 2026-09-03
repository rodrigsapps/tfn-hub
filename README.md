# TFN HUB

Script para **Steal An Egg / Roube um Ovo** (PlaceId `107778070777162`).
Interface: [WindUI](https://github.com/Footagesus/WindUI).

## Versao ofuscada (recomendada)

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/rodrigsapps/tfn-hub/refs/heads/main/cc"))()
```

## Versao limpa (codigo legivel)

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/rodrigsapps/tfn-hub/refs/heads/main/tfn"))()
```

## Recursos

- Lista de ovos com **foto 3D** (Viewport), **$/s**, **raridade** e **distancia**
- Selecionar um ovo -> o bot vai ate ele, rouba e entrega na sua base
- Auto Farm com filtros por nome, raridade e renda minima
- Desvio de guards / NPCs, anti-ragdoll
- ESP dos ovos
- Aba de diagnostico (confere remotes, plot e prompts)

## Aviso importante sobre BAN

Este jogo tem anti-cheat ativo (`ObbyAntiTPClient`, `WalkSpeedGovernor`,
`CharacterIntegrity`, `RF/RigSync/Reconcile`). Teleporte por CFrame e noclip
causam kick com o codigo **BAC-10518**.

Por isso o TFN HUB se move de forma **legitima por padrao**:
`PathfindingService` + `Humanoid:MoveTo`, respeitando a WalkSpeed que o proprio
jogo concede. Os modos **Turbo** e **Noclip** existem, mas vem **DESLIGADOS** e
sao marcados como risco.
