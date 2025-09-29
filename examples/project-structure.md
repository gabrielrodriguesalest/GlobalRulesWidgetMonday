# Estrutura de Projeto Recomendada

Este documento mostra a estrutura de projeto recomendada para Dashboard Widgets seguindo as Global Rules.

## 📁 Estrutura Completa

```
my-dashboard-widget/
├── 📁 public/
│   ├── index.html
│   ├── manifest.json
│   └── favicon.ico
├── 📁 src/
│   ├── 📁 components/           # Componentes reutilizáveis
│   │   ├── 📁 charts/          # Componentes de gráficos
│   │   │   ├── BarChart.tsx
│   │   │   ├── LineChart.tsx
│   │   │   ├── PieChart.tsx
│   │   │   └── index.ts
│   │   ├── 📁 metrics/         # Componentes de métricas
│   │   │   ├── MetricCard.tsx
│   │   │   ├── MetricsList.tsx
│   │   │   └── index.ts
│   │   ├── 📁 ui/              # Componentes de interface
│   │   │   ├── Button.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Loader.tsx
│   │   │   ├── ErrorBoundary.tsx
│   │   │   └── index.ts
│   │   └── 📁 layout/          # Componentes de layout
│   │       ├── WidgetContainer.tsx
│   │       ├── WidgetHeader.tsx
│   │       ├── WidgetContent.tsx
│   │       └── index.ts
│   ├── 📁 hooks/               # Custom hooks
│   │   ├── useWidgetData.ts    # Hook principal para dados
│   │   ├── useSettings.ts      # Hook para configurações
│   │   ├── useMonday.ts        # Hook para Monday SDK
│   │   ├── useTheme.ts         # Hook para temas
│   │   ├── usePerformance.ts   # Hook para monitoramento
│   │   └── index.ts
│   ├── 📁 services/            # Serviços e APIs
│   │   ├── mondayService.ts    # Serviço Monday API
│   │   ├── dataService.ts      # Serviço de dados
│   │   ├── cacheService.ts     # Serviço de cache
│   │   ├── validationService.ts # Serviço de validação
│   │   └── index.ts
│   ├── 📁 utils/               # Utilitários e helpers
│   │   ├── formatters.ts       # Formatadores de dados
│   │   ├── validators.ts       # Validadores
│   │   ├── calculations.ts     # Cálculos matemáticos
│   │   ├── sanitizers.ts       # Sanitização de dados
│   │   ├── accessibility.ts    # Utilitários de acessibilidade
│   │   └── index.ts
│   ├── 📁 types/               # Definições TypeScript
│   │   ├── widget.types.ts     # Tipos do widget
│   │   ├── monday.types.ts     # Tipos Monday API
│   │   ├── chart.types.ts      # Tipos de gráficos
│   │   ├── api.types.ts        # Tipos de API
│   │   └── index.ts
│   ├── 📁 constants/           # Constantes da aplicação
│   │   ├── api.constants.ts    # Constantes de API
│   │   ├── ui.constants.ts     # Constantes de UI
│   │   ├── validation.constants.ts # Constantes de validação
│   │   └── index.ts
│   ├── 📁 styles/              # Estilos globais
│   │   ├── globals.css
│   │   ├── variables.css
│   │   ├── themes.css
│   │   └── accessibility.css
│   ├── 📁 config/              # Configurações
│   │   ├── environment.ts      # Configurações de ambiente
│   │   ├── monday.config.ts    # Configurações Monday
│   │   └── widget.config.ts    # Configurações do widget
│   ├── 📁 context/             # Contextos React
│   │   ├── WidgetContext.tsx   # Contexto do widget
│   │   ├── ThemeContext.tsx    # Contexto de tema
│   │   └── index.ts
│   ├── App.tsx                 # Componente principal
│   ├── index.tsx               # Entry point
│   ├── setupTests.ts           # Configuração de testes
│   └── vite-env.d.ts          # Tipos do Vite
├── 📁 tests/                   # Testes
│   ├── 📁 __mocks__/          # Mocks para testes
│   │   ├── mondaySDK.ts
│   │   └── intersectionObserver.ts
│   ├── 📁 components/         # Testes de componentes
│   │   ├── charts/
│   │   ├── metrics/
│   │   ├── ui/
│   │   └── layout/
│   ├── 📁 hooks/              # Testes de hooks
│   ├── 📁 services/           # Testes de serviços
│   ├── 📁 utils/              # Testes de utilitários
│   ├── 📁 integration/        # Testes de integração
│   └── 📁 e2e/               # Testes E2E
│       ├── widget.spec.ts
│       └── accessibility.spec.ts
├── 📁 docs/                    # Documentação adicional
│   ├── API.md
│   ├── DEPLOYMENT.md
│   └── TROUBLESHOOTING.md
├── 📁 .github/                 # GitHub workflows
│   ├── 📁 workflows/
│   │   ├── ci.yml
│   │   ├── deploy.yml
│   │   └── accessibility.yml
│   └── 📁 ISSUE_TEMPLATE/
├── .env.example               # Exemplo de variáveis de ambiente
├── .gitignore                # Git ignore
├── .eslintrc.json            # ESLint config
├── .prettierrc               # Prettier config
├── jest.config.js            # Jest config
├── playwright.config.ts      # Playwright config
├── tsconfig.json             # TypeScript config
├── vite.config.ts            # Vite config
├── package.json              # Dependências e scripts
├── README.md                 # Documentação principal
├── CHANGELOG.md              # Histórico de mudanças
└── LICENSE                   # Licença
```

## 📋 Descrição dos Diretórios

### `/src/components/`
Componentes React reutilizáveis organizados por categoria:
- **charts/**: Componentes específicos para visualização de dados
- **metrics/**: Componentes para exibição de métricas
- **ui/**: Componentes de interface genéricos
- **layout/**: Componentes de estrutura e layout

### `/src/hooks/`
Custom hooks para lógica reutilizável:
- **useWidgetData**: Gerenciamento de dados do widget
- **useSettings**: Gerenciamento de configurações
- **useMonday**: Integração com Monday SDK
- **useTheme**: Gerenciamento de temas
- **usePerformance**: Monitoramento de performance

### `/src/services/`
Camada de serviços para comunicação externa:
- **mondayService**: Comunicação com Monday API
- **dataService**: Processamento de dados
- **cacheService**: Gerenciamento de cache
- **validationService**: Validação de dados

### `/src/utils/`
Funções utilitárias puras:
- **formatters**: Formatação de dados
- **validators**: Validação de entrada
- **calculations**: Cálculos matemáticos
- **sanitizers**: Sanitização de dados
- **accessibility**: Utilitários de acessibilidade

### `/src/types/`
Definições TypeScript organizadas por domínio:
- **widget.types**: Tipos específicos do widget
- **monday.types**: Tipos da API Monday
- **chart.types**: Tipos para gráficos
- **api.types**: Tipos de API genéricos

### `/tests/`
Testes organizados por tipo:
- **components/**: Testes unitários de componentes
- **hooks/**: Testes de custom hooks
- **services/**: Testes de serviços
- **integration/**: Testes de integração
- **e2e/**: Testes end-to-end

## 🎯 Convenções de Nomenclatura

### Arquivos
- **Componentes**: PascalCase (ex: `MetricCard.tsx`)
- **Hooks**: camelCase com prefixo `use` (ex: `useWidgetData.ts`)
- **Serviços**: camelCase com sufixo `Service` (ex: `mondayService.ts`)
- **Utilitários**: camelCase (ex: `formatters.ts`)
- **Tipos**: camelCase com sufixo `.types` (ex: `widget.types.ts`)

### Diretórios
- **kebab-case** para diretórios (ex: `chart-components/`)
- **camelCase** para arquivos TypeScript (ex: `useWidgetData.ts`)

### Exports
```typescript
// ✅ Bom - Named exports
export const MetricCard: React.FC<MetricCardProps> = ({ ... }) => { ... };
export { MetricCard };

// ✅ Bom - Index files para re-exports
export { MetricCard } from './MetricCard';
export { MetricsList } from './MetricsList';

// ❌ Evitar - Default exports para componentes
export default MetricCard;
```

## 🔧 Configurações Essenciais

### TypeScript (`tsconfig.json`)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/services/*": ["./src/services/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/types/*": ["./src/types/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Vite (`vite.config.ts`)
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@/components': resolve(__dirname, './src/components'),
      '@/hooks': resolve(__dirname, './src/hooks'),
      '@/services': resolve(__dirname, './src/services'),
      '@/utils': resolve(__dirname, './src/utils'),
      '@/types': resolve(__dirname, './src/types')
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          vibe: ['@vibe/core', '@vibe/style', '@vibe/icons'],
          monday: ['monday-sdk-js']
        }
      }
    }
  }
});
```

## 📊 Métricas de Qualidade

### Cobertura de Testes
- **Unitários**: ≥ 80% de cobertura
- **Integração**: Fluxos principais cobertos
- **E2E**: Jornadas críticas do usuário
- **Acessibilidade**: 100% compliance WCAG 2.1 AA

### Performance
- **Bundle size**: < 100KB
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 2.5s
- **Lighthouse Score**: ≥ 90

### Qualidade de Código
- **ESLint**: 0 errors, 0 warnings
- **TypeScript**: Strict mode, 100% tipagem
- **Prettier**: Formatação consistente
- **Commits**: Conventional commits

Esta estrutura garante organização, manutenibilidade e compliance com as Global Rules estabelecidas.
