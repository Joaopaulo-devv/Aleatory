<div align="center">
  <img src="banner.png" alt="Aleatory — olho de circuito em rede carmesim" width="100%">
</div>

# Aleatory

**Workspace operacional para segurança ofensiva e resposta a incidentes, conduzido por um agente de linguagem.**

O Aleatory reúne reconhecimento, auditoria local, análise de artefatos, registro de findings e documentação numa única superfície de trabalho. A interface é a **Cassandra**, um agente que opera no terminal, num painel web e num cliente desktop, com memória persistente e ferramentas que ele mesmo invoca.

> Este repositório é a apresentação do produto. O código-fonte, os binários e os pacotes não são publicados aqui e não estão disponíveis para download. Não há instruções de instalação.

---

## O que é

O Aleatory não é um scanner com uma lista fixa de verificações, nem um chatbot com um terminal colado ao lado. É uma estação de trabalho: o operador define o escopo, o agente compila o contexto do turno, escolhe e executa ferramentas, lê a saída, registra evidências e mantém o estado entre sessões.

Três ideias sustentam o desenho:

- **O operador comanda, o agente executa.** O objetivo, o alvo e os limites vêm do operador. O agente transforma isso em passos verificáveis — e para quando o resultado não confirma a hipótese.
- **Contexto é orçamento.** O prompt não carrega o catálogo inteiro de conhecimento. Ele carrega o método, um índice de skills e o que a tarefa atual exige; o resto entra sob demanda.
- **Tudo deixa rastro.** Comandos, saídas, decisões e achados ficam registrados em arquivos legíveis. A sessão pode ser retomada, auditada e exportada.

## Capacidades

| Área | O que o Aleatory faz |
| --- | --- |
| Operação assistida | Loop de agente com shell persistente, leitura de saída e correção de rota entre passos. |
| Reconhecimento | Orquestração de varreduras, fingerprint de serviços, correlação com base local de vulnerabilidades e consolidação de superfície. |
| Auditoria local | Enumeração e classificação de SUID, capabilities, sudoers, polkit, sockets de container, superfícies de persistência e permissões sensíveis — sem executar os binários auditados. |
| Análise de artefatos | Inspeção de binários, bibliotecas e pacotes; strings, símbolos, imports e layout; decompilação assistida quando o formato permite. |
| Findings | Protocolo de achado com severidade, evidência, impacto e recomendação; gravação em Markdown e deduplicação. |
| Memória operacional | Vault em Markdown com frontmatter, wikilinks, backlinks, versionamento otimista e consolidação sem perda. |
| Relatórios | Exportação de sessão e de achados em Markdown e JSON, com linha do tempo e trilha de auditoria. |
| Integração | API compatível com OpenAI para plugar os modelos configurados em editores e clientes externos. |
| Extensão | Plugins que adicionam comandos, hooks de pré e pós-execução, veto de comandos e rotas no painel. |

## Como funciona

1. **Entrada e escopo.** O operador descreve o objetivo. O agente recolhe o ambiente, o escopo declarado e o estado da sessão anterior.
2. **Compilação de contexto.** Um núcleo fixo de método é combinado com as skills relevantes ao pedido. O catálogo permanece acessível, mas só o conteúdo necessário ocupa a janela.
3. **Execução.** O agente propõe comandos e ferramentas. Com execução habilitada, roda no shell persistente e recebe a saída; sem ela, entrega os comandos como texto para revisão.
4. **Verificação.** A saída é confrontada com a hipótese. Achados só são registrados quando há evidência reproduzível; o que não fecha fica marcado como pendência.
5. **Memória e entrega.** O que importa é consolidado no vault e no relatório. A próxima sessão começa com esse estado, não do zero.

## Arquitetura

```text
┌────────────────────────────────────────────────────────────────┐
│ 1 · Interfaces    CLI · Painel web · Cliente desktop · API /v1 │
└────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 2 · Núcleo        contexto · loop · estado · verificação       │
└────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 3 · Execução      shell PTY · plugins · ferramentas externas   │
└────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────┐
│ 4 · Modelos       multi-provedor, local ou remoto              │
└────────────────────────────────────────────────────────────────┘
```

**Interfaces.** CLI, painel web e cliente desktop compartilham o mesmo núcleo. A API compatível com OpenAI expõe os modelos configurados para clientes externos, sem transformar o editor do operador num segundo executor.

**Núcleo.** O loop do agente coordena contexto, ferramentas e estado. O compilador de contexto decide o que entra no prompt a cada turno. A memória e os findings são arquivos, não caixas-pretas.

**Execução.** O shell roda em PTY persistente, então diretório, variáveis e ambiente sobrevivem entre comandos. Plugins entram no PATH do agente, podem vetar comandos antes da execução e podem expor rotas próprias no painel.

**Modelos.** A camada multi-provedor fala com endpoints compatíveis com OpenAI, com detecção por chave, rotação de credenciais, pacing, backoff e circuit breaker. Modelos locais e remotos convivem no mesmo seletor.

## Superfícies de uso

- **CLI** — operação direta no terminal, com histórico, atalhos e estatísticas de skills.
- **Painel web** — streaming de resposta, raciocínio recolhível, blocos de ferramenta, anexos, busca global, biblioteca de skills, findings e painel de memória.
- **Cliente desktop** — a mesma aplicação numa janela dedicada, com sessões, comandos de barra, permissões visíveis e exportação.
- **API** — os perfis de modelo configurados ficam disponíveis para opencode, VS Code, IDEs e qualquer cliente que fale o protocolo da OpenAI.
- **Arquivos** — memória em Markdown compatível com Obsidian, findings em Markdown, relatórios e logs em texto.

## Postura operacional

- **Escopo explícito.** Ações contra alvos remotos exigem escopo declarado. Fora dele, o agente não inicia o fluxo.
- **Execução é opt-in.** O shell começa desligado; comandos aparecem como texto até o operador habilitar a execução.
- **Salvaguardas não são opcionais por acidente.** Confirmação de comandos destrutivos, timeout por comando, veto por hook e limites de contexto são parte do núcleo.
- **Auditável por construção.** O estado é gravado em arquivos legíveis: sessões, histórico, logs de skills, findings e vault.
- **Local-first.** O workspace, a memória e os relatórios vivem na máquina do operador; o modelo é o único componente remoto, e pode ser local.

## Estado

Aleatory v1, em desenvolvimento ativo. Distribuição restrita: este repositório existe para descrever o produto, não para distribuí-lo. Não há releases, pacotes, binários ou código-fonte publicados aqui.

## Uso responsável

O Aleatory é uma ferramenta de operação em segurança. Use apenas em sistemas próprios ou em ambientes para os quais você possui autorização formal e por escrito. O operador é responsável pelo escopo, pelas ações executadas e pelos dados tratados.

## Autor

Criado e desenvolvido por **João Paulo**.
