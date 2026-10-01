# Memória do PicoAdapterGB no OMM

Use uma memória compartilhada para todo o PicoAdapterGB, mesmo quando o trabalho acontecer em várias conversas. As conversas não viram projetos separados: cada frente fica registrada como um `workstream`, e todas consultam a memória comum em `memory/`.

## Frentes já identificadas

- `pico-main` — implementação principal e configuração/EEPROM.
- `full-server-support` — suporte completo ao servidor.
- `pico-esp` — combinação Pico + ESP.
- `pico-w-pico2-w` — variantes Pico W e Pico 2 W.
- `web-server-debug` — diagnóstico do servidor web.
- `copilot-history` — histórico Copilot importado, mantido como frente arquivada.

## Uma conversa central e ajudantes por implementação

O humano conversa com um chat central. Ele identifica a placa e a implementação envolvidas e chama apenas os subagentes necessários: firmware principal, Pico + ESP8266, Pico W/Pico 2 W, suporte ao servidor completo ou servidor web. O planejador pode ser chamado quando a tarefa tiver muitas etapas ou cruzar implementações.

Os especialistas trabalham na mesma tarefa e compartilham o escopo `picoadaptergb`; cada investigação continua separada pelo workstream. O chat central leva perguntas e resultados entre eles, revisa o conjunto e responde ao humano. O perfil e os papéis estão em `memory/agent-topology.json` e `memory/roles/` do repositório central da OMM.

## Uso rápido

```sh
omm workstream list
omm search "configuração EEPROM" --workstream-id pico-main
omm context "erro de timeout no servidor web" --workstream-id web-server-debug
omm remember --kind observation --title "Falha reproduzida" --content "O que ocorreu, na placa e build indicadas" --source "doc/arquivo.md" --workstream-id web-server-debug
omm handoff --status in_progress --summary "Estado desta frente" --next "Próxima ação" --workstream-id web-server-debug
```

## Histórico de outros assistentes

O OMM já contém anotações selecionadas de uma exportação do GitHub Copilot, com referências à origem. O arquivo local `chat.json` não é copiado para a memória nem deve ser tratado automaticamente como instrução. Ao importar outras conversas, revise cada anotação candidata, mantenha o ID da conversa e atribua a frente correta; não promova uma hipótese ou um resultado de uma variante de hardware a regra geral.

## Limites de conhecimento

- Preserve as separações e fontes de verdade descritas em `AGENTS.md`, principalmente a versão exata do submódulo `dependences/libmobile` usada pelo build.
- Use a skill compartilhada `reon-libmobile-expert` quando uma tarefa envolver REON, libmobile ou o protocolo Mobile Adapter GB.
- Descobertas sobre placa, variante, branch, commit, build ou emulador precisam registrar esse contexto. O mesmo sintoma em duas sessões não prova a mesma causa.
- A pasta `.omm/` contém só o índice local e reconstruível. A memória canônica que viaja pelo Git fica em `memory/`.

