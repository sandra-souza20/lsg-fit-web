<!--
Relatório de Impacto
- Mudança de versão: 1.0.0 → 1.0.1 (PATCH: tradução para português, sem mudança de significado)
- Princípios renomeados (tradução):
  - I. Layered Architecture (Ports & Adapters) → I. Arquitetura em Camadas (Ports & Adapters)
  - II. Single Base UI on Radix → II. UI Base Única com Radix
  - III. Typography by Variants → III. Tipografia por Variantes
  - IV. Mobile-First → IV. Mobile-First
  - V. Self-Explanatory Code → V. Código Autoexplicativo
- Seções renomeadas (tradução):
  - Reference Documents → Documentos de Referência
  - Development Workflow & Quality Gates → Fluxo de Desenvolvimento e Critérios de Qualidade
  - Governance → Governança
- Seções adicionadas: nenhuma
- Seções removidas: nenhuma
- Templates que precisam de atualização: nenhum; os templates de plan/spec/tasks leem a
  constituição em tempo de execução.
- Pendências:
  - Criar docs/ com os documentos de referência iniciais (development.md, deploy.md, ...).
  - Definir como os Server Components do Next.js em src/app/ carregam dados sob o Princípio I.
-->

# Constituição do Web

## Core Principles

### I. Arquitetura em Camadas (Ports & Adapters)

Todo o código da aplicação em `src/` é organizado em camadas, e as dependências fluem em uma
só direção:

```text
app/ → components/ → hooks/ → domain/ + ports/
                                         ↑
                                       infra/ (implementa as portas)
```

- `app/` (rotas e layouts do Next.js) MUST apenas compor `components/` e `hooks/`.
- `components/` é UI pura: MUST NOT chamar `fetch`, importar bibliotecas externas (exceto o
  permitido pelo Princípio II para a UI base) nem conter regra de negócio.
- `hooks/` é a única ponte entre a UI e o resto do sistema, e o único lugar que escolhe qual
  adapter é usado. Hooks MUST consumir adapters por meio dos tipos das portas.
- `domain/` é TypeScript puro: zero React, zero biblioteca externa, zero I/O.
- `ports/` contém só contratos (interfaces e tipos), nunca implementação.
- `infra/` implementa as portas. MAY depender de `ports/` e `domain/`, e MUST NOT ser
  importada por `components/`, `domain/` ou `ports/`.
- Integrações externas (cliente HTTP, SDK do CMS, `fbq`, Graph API do Meta e similares) MUST
  ser importadas apenas dentro de `infra/`.
- Nenhuma camada depende de um adapter concreto; fora da escolha do adapter em `hooks/`, o
  código depende apenas das portas.

**Justificativa**: as regras de negócio continuam testáveis sem React e sem rede, e qualquer
fornecedor pode ser trocado escrevendo um novo adapter, sem tocar na UI nem no domínio.

### II. UI Base Única com Radix

- O projeto tem uma única biblioteca de UI base, localizada em `src/components/ui/`.
- Componentes de feature e rotas MUST NOT usar elementos HTML soltos (`div`, `button`, `p`,
  `a`, `input` etc.); MUST compor componentes da UI base.
- Os componentes da UI base MUST ser construídos sobre primitivas do Radix sempre que o Radix
  oferecer o comportamento. `src/components/ui/` é o único lugar autorizado a importar Radix e
  a renderizar HTML puro.
- O estilo visual pertence à UI base, por meio de design tokens; o Radix fornece comportamento
  e acessibilidade, nunca a aparência.

**Justificativa**: uma única fonte de verdade para comportamento, acessibilidade e identidade
visual; mudar a aparência acontece em um só lugar.

### III. Tipografia por Variantes

- Texto MUST ser renderizado pelos componentes de tipografia da UI base, escolhidos por uma
  prop `variant` (por exemplo `<Text variant="body">`, `<Heading variant="title">`).
- Classes utilitárias de fonte avulsas (tamanho, peso, família, altura de linha, espaçamento
  entre letras) MUST NOT aparecer fora dos componentes de tipografia em `src/components/ui/`.
- Novos estilos tipográficos são adicionados como novas variantes, nunca como classes avulsas.

**Justificativa**: um conjunto fechado de variantes mantém a escala tipográfica consistente e
auditável.

### IV. Mobile-First

- Os estilos MUST mirar a menor tela por padrão; telas maiores são tratadas adicionando
  modificadores de breakpoint progressivamente (`sm:`, `md:`, `lg:`, ...).
- Estilos base pensados para desktop e depois sobrescritos para mobile MUST NOT ser escritos.
- Toda feature MUST ser verificada em largura mobile antes de ser considerada pronta.

**Justificativa**: a maior parte do tráfego é mobile; desenhar a partir da menor tela evita
layouts que quebram justamente onde os usuários estão.

### V. Código Autoexplicativo

- Todo o código (identificadores, nomes de arquivos, mensagens no código) MUST ser escrito em
  inglês.
- O código MUST NOT conter comentários; a intenção é expressa por nomes e estrutura.
- Números mágicos e strings mágicas MUST NOT ser usados; valores recebem nomes como constantes
  ou design tokens.
- Todos os nomes de arquivos e diretórios MUST estar em kebab-case (por exemplo
  `use-lead-form.ts`, `meta-pixel-adapter.ts`), exceto nomes exigidos por frameworks ou
  ferramentas (por exemplo `page.tsx` e `layout.tsx` do Next.js, `README.md`).

**Justificativa**: código que se lê com clareza não diverge de comentários desatualizados, e a
nomenclatura uniforme elimina decisões arquivo a arquivo.

## Documentos de Referência

- Detalhes operacionais que podem mudar sem emenda nesta constituição ficam em `docs/`, um
  arquivo por assunto (por exemplo `docs/development.md`, `docs/deploy.md`).
- Documentos de referência MUST NOT contradizer esta constituição; em caso de conflito, a
  constituição prevalece e o documento MUST ser corrigido.
- Alterar um documento de referência não exige emenda à constituição. Mover uma regra de um
  documento de referência para esta constituição, ou o contrário, exige.
- `AGENTS.md` MUST apontar para esta constituição em `.specify/memory/constitution.md`, para
  que todo agente a leia antes de trabalhar no código.

## Fluxo de Desenvolvimento e Critérios de Qualidade

- Todo pull request MUST ser verificado contra os Princípios antes do merge; revisores
  bloqueiam mudanças que os violem.
- Os limites entre camadas (Princípio I) e as restrições de HTML solto e classes de fonte
  (Princípios II e III) SHOULD ser garantidos por regras de lint automáticas assim que
  existirem; até lá, são garantidos na revisão.
- Qualquer exceção MUST ser justificada no pull request e registrada na seção Complexity
  Tracking do plano da feature.

## Governança

- Esta constituição prevalece sobre todas as outras práticas e documentos de referência.
- Emendas são feitas via `/speckit-constitution`, revisadas em pull request, e MUST incluir uma
  nota de migração quando código existente deixar de estar em conformidade.
- O versionamento segue semver:
  - MAJOR: um princípio é removido ou redefinido de forma incompatível.
  - MINOR: um princípio ou seção é adicionado, ou a orientação é ampliada de forma relevante.
  - PATCH: esclarecimentos e ajustes de redação sem mudança de significado.
- A conformidade é verificada em todo pull request e durante o `/speckit-plan` (Constitution
  Check).

**Versão**: 1.0.1 | **Ratificada**: 2026-11-04 | **Última Emenda**: 2026-11-04
