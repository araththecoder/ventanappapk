# ventanappapk
app de ventana
## Código del cotizador

El cotizador público está en `/cotizador`. El código interactivo vive en `components/calculator/QuoteWizard.tsx`; el código siguiente muestra las reglas centrales que usa.

### Configuración comercial (`data/pricing.ts`)

```ts
/** Tarifas comerciales configurables del cotizador público (MXN). */
export const pricingConfig = {
  basePricePerSquareMeter: 4800,
  leadThreshold: 120000,
  // La instalación se cotiza después de validar ubicación y alcance.
  installationIncludedInEstimate: false,
  currency: 'MXN',
} as const;
```

### Tipos principales (`types/public-quote.ts`)

```ts
export type PublicWindow = {
  id: string;
  name: string;
  width: number;
  height: number;
  quantity: number;
  material: string;
  windowType: string;
  glass: string;
};

export type CustomerDetails = {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  city: string;
  projectType: string;
};

export type PublicQuote = {
  items: PublicWindow[];
  totalSquareMeters: number;
  subtotal: number;
  installation: number | null;
  estimatedTotal: number;
};
```

### Motor de precio (`lib/public-pricing.ts`)

```ts
import { pricingConfig } from '@/data/pricing';
import type { PublicQuote, PublicWindow } from '@/types/public-quote';

export function calculatePublicQuote(items: PublicWindow[]): PublicQuote {
  const totalSquareMeters = items.reduce(
    (sum, item) => sum + item.width * item.height * item.quantity,
    0,
  );
  const subtotal = totalSquareMeters * pricingConfig.basePricePerSquareMeter;

  return {
    items,
    totalSquareMeters,
    subtotal,
    installation: pricingConfig.installationIncludedInEstimate ? 0 : null,
    estimatedTotal: subtotal,
  };
}

export function isValidEmail(value: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/.test(value.trim());
}

export function isValidPhone(value: string): boolean {
  const digits = value.replace(/\D/g, '');
  return digits.length >= 10 && digits.length <= 15;
}
```

### Catálogos (`data/quote-options.ts`)

```ts
export const materials = [
  { id: 'pvc', name: 'PVC' },
  { id: 'aluminio', name: 'Aluminio' },
] as const;

export const windowTypes = [
  { id: 'fija', name: 'Fija' },
  { id: 'corrediza', name: 'Corrediza' },
  { id: 'abatible', name: 'Abatible' },
  { id: 'puerta', name: 'Puerta' },
] as const;

// El vidrio todavía no tiene incrementos de precio configurados.
export const glassOptions = [
  { id: 'claro-6mm', name: 'Claro 6 mm' },
  { id: 'claro-9mm', name: 'Claro 9 mm' },
  { id: 'laminado', name: 'Laminado' },
  { id: 'doble', name: 'Doble vidrio' },
] as const;
```

### Archivos principales

```text
app/page.tsx                         Portada y CTA “Cotiza tus ventanas”
app/cotizador/page.tsx               Ruta del cotizador
components/calculator/QuoteWizard.tsx Flujo de ventanas, resumen y formulario
data/pricing.ts                      Tarifa base y umbral comercial
data/quote-options.ts                Materiales, tipos y vidrios
lib/public-pricing.ts                Superficie, precio y validaciones
types/public-quote.ts                Tipos del cotizador público
app/operaciones/page.tsx             Dashboard interno existente
```

Los ejemplos de este README documentan el código fuente y deben mantenerse sincronizados con esos archivos al cambiar reglas o precios.

La tarifa de referencia inicial es $4,800 MXN por m² y el umbral para atención personalizada es $120,000 MXN; ambos se ajustan en `data/pricing.ts`. La selección de vidrio se guarda en el resumen, pero todavía no aplica un incremento. La instalación se cotiza por separado.
