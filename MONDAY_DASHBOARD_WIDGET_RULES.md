# Monday.com Dashboard Widgets - Global Rules

**Versão**: 1.0  
**Data**: 2024-12-29  
**Autor**: Sistema de Documentação Especializado  
**Status**: Ativo  
**Tags**: `monday-widgets`, `dashboard`, `development`, `global-rules`, `best-practices`

---

## 📋 Sumário Executivo

Este documento estabelece as **Global Rules** específicas para desenvolvimento de **Dashboard Widgets** na plataforma Monday.com, baseado nas regras organizacionais existentes e focado exclusivamente nas particularidades técnicas e de design de widgets de dashboard.

### 🎯 Objetivo
Estabelecer um padrão consistente e robusto para o desenvolvimento de widgets de dashboard que garantam qualidade, performance, segurança e experiência do usuário excepcional.

### 🏗️ Escopo
- **Foco Exclusivo**: Dashboard Widgets (não Views)
- **Arquitetura**: React + TypeScript + Monday SDK
- **Design**: Monday Vibe Design System
- **Performance**: Otimização para múltiplas boards
- **Segurança**: Validação e sanitização robusta
- **Acessibilidade**: WCAG 2.1 AA compliance

---

## 📚 Estrutura da Documentação

### 📋 Documento Principal
- **[MONDAY DASHBOARD WIDGET RULES](./MONDAY_DASHBOARD_WIDGET_RULES.md)** - Este documento com todas as regras globais

### 🏗️ Fundamentos e Arquitetura
- **[Architecture Patterns](./architecture-patterns.md)** - Padrões arquiteturais específicos para widgets
- **[Development Standards](./development-standards.md)** - Padrões de desenvolvimento e código
- **[Component Structure](./component-structure.md)** - Estrutura e organização de componentes

### 🎨 Design e UX
- **[Design System Integration](./design-system.md)** - Integração com Monday Vibe Design System
- **[Responsive Design](./responsive-design.md)** - Responsividade e adaptação de tamanhos
- **[Accessibility Standards](./accessibility-standards.md)** - Padrões de acessibilidade WCAG 2.1 AA

### ⚡ Performance e Otimização
- **[Performance Optimization](./performance-optimization.md)** - Otimização de performance para widgets
- **[Data Management](./data-management.md)** - Gerenciamento eficiente de dados
- **[Caching Strategies](./caching-strategies.md)** - Estratégias de cache e otimização

### 🔒 Segurança e Compliance
- **[Security Guidelines](./security-guidelines.md)** - Diretrizes de segurança específicas
- **[Data Validation](./data-validation.md)** - Validação e sanitização de dados
- **[Compliance Standards](./compliance-standards.md)** - Padrões de compliance (LGPD/GDPR)

### 🧪 Qualidade e Testes
- **[Testing Strategies](./testing-strategies.md)** - Estratégias de teste para widgets
- **[Quality Assurance](./quality-assurance.md)** - Garantia de qualidade
- **[Error Handling](./error-handling.md)** - Tratamento de erros robusto

### 🚀 Deploy e Produção
- **[Deployment Guidelines](./deployment-guidelines.md)** - Diretrizes de deploy
- **[Monitoring & Observability](./monitoring-observability.md)** - Monitoramento e observabilidade
- **[Maintenance Procedures](./maintenance-procedures.md)** - Procedimentos de manutenção

---

## 🎯 Global Rules para Dashboard Widgets

## 1. Arquitetura e Desenvolvimento

### 1.1 Tecnologias Obrigatórias
```typescript
// Stack tecnológico obrigatório
const REQUIRED_STACK = {
  framework: 'React 18+',
  language: 'TypeScript 5+',
  sdk: 'monday-sdk-js (latest)',
  designSystem: '@vibe/core',
  bundler: 'Vite 5+ ou Webpack 5+',
  testing: 'Jest + React Testing Library'
};
```

### 1.2 Estrutura de Projeto Padrão
```
src/
├── components/           # Componentes reutilizáveis
│   ├── charts/          # Componentes de gráficos
│   ├── metrics/         # Componentes de métricas
│   ├── ui/              # Componentes de interface
│   └── layout/          # Componentes de layout
├── hooks/               # Custom hooks
│   ├── useWidgetData.ts # Hook principal para dados
│   ├── useSettings.ts   # Hook para configurações
│   └── useMonday.ts     # Hook para Monday SDK
├── services/            # Serviços e APIs
│   ├── mondayService.ts # Serviço Monday API
│   ├── dataService.ts   # Serviço de dados
│   └── cacheService.ts  # Serviço de cache
├── utils/               # Utilitários e helpers
│   ├── formatters.ts    # Formatadores de dados
│   ├── validators.ts    # Validadores
│   └── calculations.ts  # Cálculos matemáticos
├── types/               # Definições TypeScript
│   ├── widget.types.ts  # Tipos do widget
│   ├── monday.types.ts  # Tipos Monday API
│   └── chart.types.ts   # Tipos de gráficos
├── constants/           # Constantes da aplicação
├── styles/              # Estilos globais
└── tests/               # Testes unitários
```

### 1.3 Componentização Modular Obrigatória
```typescript
// Estrutura base obrigatória para todos os widgets
interface WidgetProps {
  settings: WidgetSettings;
  boardIds: string[];
  size: WidgetSize;
  onSettingsChange?: (settings: WidgetSettings) => void;
}

const BaseWidget: React.FC<WidgetProps> = ({
  settings,
  boardIds,
  size,
  onSettingsChange
}) => {
  // Implementação base obrigatória
};
```

## 2. Design e Interface

### 2.1 Monday Vibe Design System (Obrigatório)
```typescript
// Importações obrigatórias do Vibe
import { 
  Button, 
  TextField, 
  Flex, 
  Box,
  Text,
  Loader,
  Icon
} from '@vibe/core';

// Uso de tokens de design obrigatório
import { spacing, colors, typography } from '@vibe/style';
```

### 2.2 Responsividade Obrigatória
```typescript
// Tamanhos de widget obrigatórios
type WidgetSize = 'small' | 'medium' | 'large';

const WIDGET_DIMENSIONS = {
  small: { width: 300, height: 200 },
  medium: { width: 600, height: 400 },
  large: { width: 900, height: 600 }
} as const;
```

### 2.3 Temas Claro e Escuro (Obrigatório)
```typescript
// Suporte obrigatório a temas
const useTheme = () => {
  const { theme } = useMondayContext();
  return theme === 'dark' ? darkTheme : lightTheme;
};
```

## 3. Performance

### 3.1 Tempo de Carregamento (Obrigatório)
- **Tempo máximo**: 2 segundos para carregamento inicial
- **Lazy loading**: Obrigatório para componentes pesados
- **Code splitting**: Obrigatório para widgets complexos

### 3.2 Otimização de Renderização (Obrigatório)
```typescript
// Memoização obrigatória
const OptimizedWidget = memo(({ data, settings }) => {
  const processedData = useMemo(() => 
    processWidgetData(data), [data]
  );
  
  const handleAction = useCallback((action) => {
    // Handle action
  }, []);
  
  return <WidgetContent />;
});
```

### 3.3 Limitação de Chamadas API (Obrigatório)
```typescript
// Limites obrigatórios
const API_LIMITS = {
  maxBoardsPerWidget: 10,
  maxItemsPerBoard: 1000,
  refreshInterval: 30000, // 30 segundos mínimo
  maxConcurrentRequests: 3
};
```

## 4. Segurança

### 4.1 Validação de Dados (Obrigatória)
```typescript
// Schema de validação obrigatório
import Joi from 'joi';

const widgetDataSchema = Joi.object({
  boardIds: Joi.array().items(Joi.string()).max(10).required(),
  settings: Joi.object().required(),
  data: Joi.array().max(1000).required()
});

// Validação obrigatória em todos os inputs
const validateWidgetData = (data: unknown) => {
  const { error, value } = widgetDataSchema.validate(data);
  if (error) throw new ValidationError(error.message);
  return value;
};
```

### 4.2 Sanitização Obrigatória
```typescript
// Sanitização obrigatória de HTML
import DOMPurify from 'dompurify';

const sanitizeHtml = (html: string) => 
  DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
    ALLOWED_ATTR: []
  });
```

### 4.3 Environment Variables (Obrigatório)
```typescript
// Configuração segura obrigatória
const config = {
  apiUrl: process.env.MONDAY_API_URL,
  apiToken: process.env.MONDAY_API_TOKEN, // Nunca hardcoded
  environment: process.env.NODE_ENV
};
```

## 5. Acessibilidade

### 5.1 WCAG 2.1 AA Compliance (Obrigatório)
```typescript
// Estrutura acessível obrigatória
const AccessibleWidget = ({ title, description, data }) => {
  const titleId = useId();
  const descId = useId();
  
  return (
    <div
      role="img"
      aria-labelledby={titleId}
      aria-describedby={descId}
      tabIndex={0}
    >
      <h3 id={titleId}>{title}</h3>
      <p id={descId} className="sr-only">{description}</p>
      {/* Conteúdo do widget */}
    </div>
  );
};
```

### 5.2 Navegação por Teclado (Obrigatória)
```typescript
// Suporte obrigatório a navegação por teclado
const handleKeyDown = (event: KeyboardEvent) => {
  switch (event.key) {
    case 'Enter':
    case ' ':
      // Ativar elemento
      break;
    case 'Escape':
      // Fechar modal/dropdown
      break;
  }
};
```

## 6. Integração Monday

### 6.1 Monday SDK Usage (Obrigatório)
```typescript
// Inicialização obrigatória
import mondaySdk from 'monday-sdk-js';

const monday = mondaySdk();

// Context obrigatório
const useMondayContext = () => {
  const [context, setContext] = useState(null);
  
  useEffect(() => {
    monday.get('context').then(setContext);
  }, []);
  
  return context;
};
```

### 6.2 Rate Limiting (Obrigatório)
```typescript
// Rate limiting obrigatório
const rateLimiter = new RateLimiter({
  tokensPerInterval: 100,
  interval: 'minute'
});

const executeQuery = async (query: string) => {
  await rateLimiter.removeTokens(1);
  return monday.api(query);
};
```

### 6.3 Error Handling (Obrigatório)
```typescript
// Error boundary obrigatório
class WidgetErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Widget Error:', error, errorInfo);
    // Log para serviço de monitoramento
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

## 7. Testes e Qualidade

### 7.1 Cobertura de Testes (Obrigatória)
- **Testes unitários**: Mínimo 80% de cobertura
- **Testes de integração**: Obrigatórios para fluxos principais
- **Testes de acessibilidade**: 100% compliance

### 7.2 Estrutura de Testes (Obrigatória)
```typescript
// Estrutura de teste obrigatória
describe('WidgetComponent', () => {
  test('renders with valid data', () => {
    render(<WidgetComponent data={mockData} />);
    expect(screen.getByRole('img')).toBeInTheDocument();
  });
  
  test('handles loading state', () => {
    render(<WidgetComponent loading={true} />);
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });
  
  test('handles error state', () => {
    render(<WidgetComponent error="Test error" />);
    expect(screen.getByRole('alert')).toBeInTheDocument();
  });
});
```

## 8. Documentação

### 8.1 Documentação Obrigatória
- **README.md**: Detalhado com setup e uso
- **CHANGELOG.md**: Histórico de versões
- **API.md**: Documentação da API do widget
- **CONTRIBUTING.md**: Guia de contribuição

### 8.2 Comentários de Código (Obrigatórios)
```typescript
/**
 * Widget para exibição de métricas de dashboard
 * @param props - Propriedades do widget
 * @param props.boardIds - IDs das boards para agregação
 * @param props.settings - Configurações do widget
 * @returns Componente React do widget
 */
const MetricsWidget: React.FC<MetricsWidgetProps> = ({
  boardIds,
  settings
}) => {
  // Implementação
};
```

## 9. Compliance e Boas Práticas

### 9.1 Princípios SOLID (Obrigatórios)
- **Single Responsibility**: Cada componente tem uma responsabilidade
- **Open/Closed**: Extensível mas fechado para modificação
- **Liskov Substitution**: Substituição sem quebrar funcionalidade
- **Interface Segregation**: Interfaces específicas e focadas
- **Dependency Inversion**: Dependência de abstrações

### 9.2 Code Quality (Obrigatório)
```json
// ESLint configuration obrigatória
{
  "extends": [
    "@typescript-eslint/recommended",
    "react-hooks/recommended"
  ],
  "rules": {
    "no-console": "error",
    "prefer-const": "error",
    "@typescript-eslint/no-unused-vars": "error"
  }
}
```

## 10. Monitoramento e Logs

### 10.1 Logging Estruturado (Obrigatório)
```typescript
// Logger estruturado obrigatório
interface LogEntry {
  level: 'error' | 'warn' | 'info' | 'debug';
  message: string;
  timestamp: string;
  widgetId: string;
  userId?: string;
  context?: Record<string, any>;
}

const logger = {
  error: (message: string, context?: Record<string, any>) => {
    const entry: LogEntry = {
      level: 'error',
      message,
      timestamp: new Date().toISOString(),
      widgetId: getWidgetId(),
      userId: getCurrentUserId(),
      context
    };
    console.error(JSON.stringify(entry));
  }
};
```

### 10.2 Métricas de Performance (Obrigatórias)
```typescript
// Métricas obrigatórias
const trackPerformance = (metric: string, value: number) => {
  console.log(`Metric: ${metric} = ${value}ms`, {
    timestamp: new Date().toISOString(),
    widgetId: getWidgetId(),
    sessionId: getSessionId()
  });
};
```

---

## 📊 Checklists de Compliance

### ✅ Checklist de Desenvolvimento
- [ ] Estrutura de projeto seguindo padrão obrigatório
- [ ] TypeScript com tipagem forte implementada
- [ ] Monday Vibe Design System integrado
- [ ] Responsividade para todos os tamanhos implementada
- [ ] Temas claro e escuro suportados
- [ ] Performance otimizada (< 2s carregamento)
- [ ] Memoização implementada onde necessário
- [ ] Rate limiting configurado

### ✅ Checklist de Segurança
- [ ] Validação de dados implementada
- [ ] Sanitização de HTML implementada
- [ ] Environment variables configuradas
- [ ] Tokens de API protegidos
- [ ] Headers de segurança configurados
- [ ] Logs de auditoria implementados

### ✅ Checklist de Acessibilidade
- [ ] WCAG 2.1 AA compliance verificada
- [ ] Navegação por teclado funcional
- [ ] ARIA labels implementados
- [ ] Contraste de cores adequado
- [ ] Suporte a leitores de tela
- [ ] Testes de acessibilidade automatizados

### ✅ Checklist de Qualidade
- [ ] Cobertura de testes ≥ 80%
- [ ] Testes de integração implementados
- [ ] ESLint sem erros
- [ ] Prettier configurado
- [ ] Documentação completa
- [ ] Error boundaries implementados

### ✅ Checklist de Deploy
- [ ] Build otimizado sem warnings
- [ ] Bundle size analisado e otimizado
- [ ] Versionamento semântico aplicado
- [ ] Changelog atualizado
- [ ] Monitoramento configurado
- [ ] Rollback plan definido

---

## 🔗 Recursos e Referências

### 📚 Documentação Oficial
- [Monday.com Developer Center](https://developer.monday.com/apps)
- [Monday SDK Documentation](https://developer.monday.com/apps/docs/introduction-to-the-sdk)
- [Vibe Design System](https://vibe.monday.com/)
- [GraphQL API Reference](https://developer.monday.com/api-reference)

### 🛠️ Ferramentas Recomendadas
- **Development**: Vite, TypeScript, ESLint, Prettier
- **Testing**: Jest, React Testing Library, Playwright
- **Monitoring**: Sentry, LogRocket, Lighthouse CI
- **Performance**: Webpack Bundle Analyzer, React DevTools Profiler

### 📖 Padrões e Metodologias
- **Architecture**: Clean Architecture, SOLID Principles
- **Testing**: Testing Trophy, AAA Pattern
- **Accessibility**: WCAG 2.1 Guidelines
- **Performance**: Core Web Vitals, RAIL Model

---

## 📝 Versionamento e Atualizações

- **Versão atual**: 1.0 (Dezembro 2024)
- **Próxima revisão**: Março 2025
- **Responsável**: Equipe de Arquitetura

> **Nota**: Esta documentação segue os padrões estabelecidos nas Global Rules organizacionais e é mantida atualizada conforme as evoluções da plataforma Monday.com e melhores práticas da indústria.

---

**© 2024 - Monday.com Dashboard Widgets Global Rules**
