# Setup do Repositório - Monday Dashboard Widgets Global Rules

Este guia explica como configurar e usar este repositório de Global Rules.

## 🚀 Quick Start

### 1. Clone o Repositório
```bash
git clone https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday.git
cd GlobalRulesWidgetMonday
```

### 2. Instale as Dependências
```bash
npm install
```

### 3. Valide a Documentação
```bash
npm run validate
```

### 4. Sirva Localmente (Opcional)
```bash
npm run serve
# Acesse http://localhost:8080
```

## 📋 Comandos Disponíveis

### Validação
```bash
# Validar toda a documentação
npm run validate

# Validar apenas Markdown
npm run lint:md

# Validar apenas links
npm run lint:links
```

### Desenvolvimento
```bash
# Servir documentação localmente
npm run serve

# Build da documentação
npm run build:docs
```

### Verificações
```bash
# Verificar acessibilidade
npm run check:accessibility

# Verificar performance
npm run check:performance
```

## 📁 Estrutura do Repositório

```
GlobalRulesWidgetMonday/
├── 📋 README.md                           # Índice principal e navegação
├── 📋 MONDAY_DASHBOARD_WIDGET_RULES.md    # Documento principal com todas as regras
├── 🏗️ architecture-patterns.md            # Padrões arquiteturais específicos
├── 💻 development-standards.md            # Padrões de desenvolvimento e código
├── 🎨 design-system.md                    # Integração com Monday Vibe Design System
├── ⚡ performance-optimization.md         # Otimização de performance para widgets
├── 🔒 security-guidelines.md              # Diretrizes de segurança específicas
├── ♿ accessibility-standards.md          # Padrões de acessibilidade WCAG 2.1 AA
├── 🧪 testing-strategies.md               # Estratégias de teste para widgets
├── 🚀 deployment-guidelines.md            # Diretrizes de deploy e monitoramento
├── 📝 CHANGELOG.md                        # Histórico de versões
├── 🤝 CONTRIBUTING.md                     # Guia de contribuição
├── 📄 LICENSE                             # Licença MIT
├── 📦 package.json                        # Configuração do projeto
├── 🔧 .markdownlint.json                  # Configuração do linter
├── 🚫 .gitignore                          # Arquivos ignorados pelo Git
├── 📁 .github/                            # Templates e workflows
│   ├── workflows/validate-docs.yml       # CI/CD para validação
│   ├── ISSUE_TEMPLATE/                    # Templates de issues
│   └── pull_request_template.md          # Template de PR
└── 📁 examples/                           # Exemplos e templates
    ├── widget-config-example.json        # Configuração exemplo
    └── project-structure.md              # Estrutura de projeto recomendada
```

## 🎯 Como Usar as Global Rules

### Para Novos Projetos
1. **Leia o [README.md](./README.md)** para visão geral
2. **Consulte [MONDAY_DASHBOARD_WIDGET_RULES.md](./MONDAY_DASHBOARD_WIDGET_RULES.md)** para regras principais
3. **Use [examples/project-structure.md](./examples/project-structure.md)** como base
4. **Configure seu projeto** com [examples/widget-config-example.json](./examples/widget-config-example.json)

### Para Projetos Existentes
1. **Faça auditoria** usando as regras como checklist
2. **Implemente gradualmente** as melhorias necessárias
3. **Priorize** segurança e acessibilidade
4. **Monitore** performance e qualidade

### Para Equipes
1. **Treine desenvolvedores** nas Global Rules
2. **Integre** validações no CI/CD
3. **Estabeleça** code review baseado nas regras
4. **Monitore** compliance continuamente

## 📊 Checklist de Compliance

### ✅ Desenvolvimento
- [ ] React 18+ + TypeScript 5+
- [ ] Monday Vibe Design System
- [ ] Estrutura de projeto padrão
- [ ] Custom hooks implementados
- [ ] Validação de dados robusta

### ✅ Performance
- [ ] Bundle size < 100KB
- [ ] Tempo de carregamento < 2s
- [ ] Lazy loading implementado
- [ ] Memoização otimizada
- [ ] Cache multi-nível

### ✅ Segurança
- [ ] Validação com Joi/Zod
- [ ] Sanitização HTML
- [ ] Rate limiting configurado
- [ ] Environment variables seguras
- [ ] CSP headers implementados

### ✅ Acessibilidade
- [ ] WCAG 2.1 AA compliance
- [ ] Navegação por teclado
- [ ] Screen readers suportados
- [ ] Contraste adequado (4.5:1)
- [ ] ARIA labels implementados

### ✅ Testes
- [ ] Cobertura ≥ 80%
- [ ] Testes unitários (Jest + RTL)
- [ ] Testes de integração
- [ ] Testes E2E (Playwright)
- [ ] Testes de acessibilidade

### ✅ Deploy
- [ ] CI/CD configurado
- [ ] Health checks implementados
- [ ] Monitoramento ativo
- [ ] Estratégia de rollback
- [ ] Performance monitoring

## 🛠️ Ferramentas Recomendadas

### IDEs e Extensões
- **VSCode** com extensões:
  - ES7+ React/Redux/React-Native snippets
  - TypeScript Importer
  - ESLint
  - Prettier
  - Auto Rename Tag
  - Bracket Pair Colorizer
  - GitLens

### Desenvolvimento
- **Node.js**: ≥ 18.0.0
- **npm**: ≥ 9.0.0
- **Git**: Para controle de versão
- **Chrome DevTools**: Para debugging

### Testes
- **Jest**: Testes unitários
- **React Testing Library**: Testes de componentes
- **Playwright**: Testes E2E
- **axe-core**: Testes de acessibilidade

### Monitoramento
- **Lighthouse**: Performance e acessibilidade
- **Sentry**: Error tracking
- **LogRocket**: Session replay
- **Bundle Analyzer**: Análise de bundle

## 🔄 Processo de Atualização

### Quando Atualizar
- **Mensalmente**: Verificar atualizações das Global Rules
- **Por release**: Validar compliance antes do deploy
- **Por mudança**: Revisar impacto nas regras

### Como Atualizar
1. **Pull** das últimas mudanças
2. **Leia** o [CHANGELOG.md](./CHANGELOG.md)
3. **Identifique** breaking changes
4. **Atualize** seu projeto conforme necessário
5. **Teste** todas as funcionalidades
6. **Deploy** com confiança

## 📞 Suporte

### Documentação
- **[README.md](./README.md)**: Visão geral e navegação
- **[CONTRIBUTING.md](./CONTRIBUTING.md)**: Como contribuir
- **[examples/](./examples/)**: Exemplos práticos

### Comunidade
- **[Issues](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/issues)**: Reportar bugs ou sugerir melhorias
- **[Discussions](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/discussions)**: Discussões gerais
- **[Pull Requests](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/pulls)**: Contribuições

### Recursos Externos
- **[Monday.com Developer Center](https://developer.monday.com/)**
- **[Vibe Design System](https://vibe.monday.com/)**
- **[React Documentation](https://react.dev/)**
- **[TypeScript Handbook](https://www.typescriptlang.org/docs/)**

---

**🎉 Pronto para começar! Siga as Global Rules e desenvolva widgets de dashboard excepcionais!**
