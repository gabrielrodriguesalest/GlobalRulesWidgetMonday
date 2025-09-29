# Monday Dashboard Widgets - Global Rules

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Documentation](https://img.shields.io/badge/docs-latest-brightgreen.svg)](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/releases)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

> **Documentação especializada para desenvolvimento de Dashboard Widgets na plataforma Monday.com**

🔗 **Repositório**: [https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday)

## 📋 Visão Geral

Esta documentação estabelece as **Global Rules** específicas para desenvolvimento de **Dashboard Widgets** na plataforma Monday.com, baseada nas regras organizacionais existentes e focada exclusivamente nas particularidades técnicas e de design de widgets de dashboard.

### 🎯 Objetivo Principal
Estabelecer um padrão consistente e robusto para o desenvolvimento de widgets de dashboard que garantam:
- **Qualidade**: Código limpo e manutenível
- **Performance**: Carregamento rápido e responsivo
- **Segurança**: Proteção de dados e validação robusta
- **Acessibilidade**: Compliance WCAG 2.1 AA
- **Experiência do Usuário**: Interface intuitiva e consistente

## 📚 Estrutura da Documentação

### 📋 Documento Principal
- **[MONDAY_DASHBOARD_WIDGET_RULES.md](./MONDAY_DASHBOARD_WIDGET_RULES.md)** - Documento principal com todas as regras globais

### 📁 Documentação Especializada (pasta `docs/`)
- **[architecture-patterns.md](./docs/architecture-patterns.md)** - Padrões arquiteturais específicos
- **[development-standards.md](./docs/development-standards.md)** - Padrões de desenvolvimento
- **[design-system.md](./docs/design-system.md)** - Integração com Monday Vibe Design System
- **[performance-optimization.md](./docs/performance-optimization.md)** - Otimização específica para widgets
- **[security-guidelines.md](./docs/security-guidelines.md)** - Diretrizes de segurança robustas
- **[accessibility-standards.md](./docs/accessibility-standards.md)** - Padrões WCAG 2.1 AA compliance
- **[testing-strategies.md](./docs/testing-strategies.md)** - Estratégias de teste para widgets
- **[deployment-guidelines.md](./docs/deployment-guidelines.md)** - Diretrizes de deploy e monitoramento

## 🚀 Instalação e Setup

### 1. Clone o Repositório
```bash
git clone https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday.git
cd GlobalRulesWidgetMonday
```

### 2. Instale Dependências (Opcional)
```bash
npm install
```

### 3. Valide a Documentação
```bash
npm run validate
```

### 4. Sirva Localmente
```bash
npm run serve
# Acesse http://localhost:8080
```

## 🚀 Quick Start

### 1. Configuração Inicial
```bash
# Criar novo projeto de widget
npx create-monday-app my-dashboard-widget --template=widget

# Instalar dependências obrigatórias
npm install @vibe/core @vibe/style @vibe/icons
npm install monday-sdk-js typescript

# Instalar dependências de desenvolvimento
npm install --save-dev @types/react @types/node
npm install --save-dev eslint prettier jest
```

### 2. Estrutura Base
```typescript
// src/App.tsx
import React from 'react';
import { VibeProvider } from '@vibe/core';
import { DashboardWidget } from './components/DashboardWidget';

const App: React.FC = () => {
  return (
    <VibeProvider>
      <DashboardWidget />
    </VibeProvider>
  );
};

export default App;
```

### 3. Widget Base
```typescript
// src/components/DashboardWidget.tsx
import React, { useState, useEffect } from 'react';
import { Box, Text, Loader } from '@vibe/core';
import { useWidgetData } from '../hooks/useWidgetData';

interface DashboardWidgetProps {
  boardIds: string[];
  settings: WidgetSettings;
  size: WidgetSize;
}

const DashboardWidget: React.FC<DashboardWidgetProps> = ({
  boardIds,
  settings,
  size
}) => {
  const { data, loading, error } = useWidgetData(boardIds);
  
  if (loading) return <Loader />;
  if (error) return <Text>Error: {error.message}</Text>;
  
  return (
    <Box>
      <Text>{settings.title}</Text>
      {/* Widget content */}
    </Box>
  );
};

export default DashboardWidget;
```

## 📊 Principais Diferenças: Views vs Widgets

| Aspecto | Views | Dashboard Widgets |
|---------|-------|-------------------|
| **Contexto** | Uma única board | Múltiplas boards (dashboard) |
| **Localização** | Abas sob o título da board | Gadgets no dashboard |
| **Dados** | Até 10.000 itens por board | Até 20.000 itens por widget |
| **Complexidade** | Menor (escopo único) | Maior (agregação de dados) |
| **Responsividade** | Fixa ao container da board | Adaptável (small/medium/large) |
| **Cache** | Simples (board específico) | Complexo (múltiplas fontes) |

## 🎯 Regras Globais Essenciais

### 1. Tecnologias Obrigatórias
- **Framework**: React 18+ com TypeScript 5+
- **Design System**: Monday Vibe (@vibe/core)
- **SDK**: monday-sdk-js (versão mais recente)
- **Bundler**: Vite 5+ ou Webpack 5+
- **Testes**: Jest + React Testing Library

### 2. Performance Obrigatória
- **Tempo de carregamento**: Máximo 2 segundos
- **Bundle size**: Máximo 100KB
- **Memoização**: Obrigatória para cálculos pesados
- **Lazy loading**: Obrigatório para componentes pesados

### 3. Segurança Obrigatória
- **Validação**: Todos os inputs devem ser validados
- **Sanitização**: HTML deve ser sanitizado
- **Rate limiting**: Máximo 100 requests/minuto
- **Environment variables**: Nunca hardcode tokens

### 4. Acessibilidade Obrigatória
- **WCAG 2.1 AA**: Compliance total
- **Navegação por teclado**: Suporte completo
- **Screen readers**: Compatibilidade total
- **Contraste**: Mínimo 4.5:1

## 🛠️ Ferramentas Recomendadas

### Desenvolvimento
- **IDE**: VSCode com extensões React/TypeScript
- **Linting**: ESLint + Prettier + Husky
- **Testing**: Jest + React Testing Library + Playwright
- **Bundling**: Vite com plugins de otimização

### Monitoramento
- **Performance**: Lighthouse CI, Web Vitals
- **Errors**: Sentry, LogRocket
- **Analytics**: Monday Analytics API
- **Bundle Analysis**: webpack-bundle-analyzer

### Design
- **Figma**: Para mockups e protótipos
- **Storybook**: Para documentação de componentes
- **Chromatic**: Para testes visuais
- **Accessibility**: axe-core, WAVE

## 📈 Métricas de Qualidade

### Performance Targets
- **First Contentful Paint**: < 1.5s
- **Largest Contentful Paint**: < 2.0s
- **Time to Interactive**: < 2.5s
- **Cumulative Layout Shift**: < 0.1

### Code Quality Targets
- **Test Coverage**: ≥ 80%
- **TypeScript Coverage**: 100%
- **ESLint Errors**: 0
- **Bundle Size**: < 100KB

### Accessibility Targets
- **WCAG 2.1 AA**: 100% compliance
- **Keyboard Navigation**: 100% functional
- **Screen Reader**: 100% compatible
- **Color Contrast**: ≥ 4.5:1

## 🔄 Processo de Desenvolvimento

### 1. Planejamento
- [ ] Definir requisitos funcionais
- [ ] Criar wireframes e mockups
- [ ] Definir estrutura de dados
- [ ] Planejar testes

### 2. Desenvolvimento
- [ ] Configurar projeto base
- [ ] Implementar componentes core
- [ ] Integrar com Monday SDK
- [ ] Implementar testes

### 3. Qualidade
- [ ] Executar testes automatizados
- [ ] Validar acessibilidade
- [ ] Otimizar performance
- [ ] Revisar segurança

### 4. Deploy
- [ ] Build de produção
- [ ] Testes de integração
- [ ] Deploy para staging
- [ ] Deploy para produção

## 📞 Suporte e Contribuição

### Canais de Suporte
- **📋 Documentação**: Esta documentação
- **🌐 Monday Developer Center**: [developer.monday.com](https://developer.monday.com)
- **💬 Community**: Monday Developer Community
- **🐛 Issues**: [GitHub Issues](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/issues)
- **💡 Discussions**: [GitHub Discussions](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/discussions)

### Como Contribuir
1. **Fork** o repositório
2. **Leia** o [CONTRIBUTING.md](./CONTRIBUTING.md)
3. **Siga** os [Development Standards](./development-standards.md)
4. **Implemente** testes adequados
5. **Documente** mudanças
6. **Submeta** pull request

### 🔗 Links Importantes
- **📦 Repositório**: [GitHub](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday)
- **📋 Issues**: [Reportar Bugs](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/issues/new?template=bug_report.md)
- **✨ Feature Requests**: [Sugerir Melhorias](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/issues/new?template=feature_request.md)
- **📖 Wiki**: [Documentação Adicional](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/wiki)
- **🚀 Releases**: [Changelog](https://github.com/gabrielrodriguesalest/GlobalRulesWidgetMonday/releases)

## 📝 Changelog

### Versão 1.0 (Dezembro 2024)
- ✅ Criação da documentação inicial
- ✅ Definição de regras globais
- ✅ Padrões de arquitetura
- ✅ Guidelines de segurança
- ✅ Otimizações de performance

### Próximas Versões
- 🔄 Integração com Monday AI
- 🔄 Suporte a Web Components
- 🔄 Melhorias de acessibilidade
- 🔄 Novos padrões de design

---

## 🏆 Certificação de Compliance

Este projeto segue as **Global Rules** estabelecidas e mantém compliance com:

- ✅ **Monday.com Platform Standards**
- ✅ **WCAG 2.1 AA Accessibility**
- ✅ **LGPD/GDPR Data Protection**
- ✅ **Security Best Practices**
- ✅ **Performance Standards**

---

**© 2024 - Monday.com Dashboard Widgets Global Rules**  
*Documentação mantida pela equipe de Arquitetura e atualizada conforme evoluções da plataforma Monday.com*
