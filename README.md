# 🌎 LATAM Freelance Tax & Payment AI Context Layer

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Language](https://img.shields.io/badge/language-ES%20%7C%20EN-green.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()
[![Ko-fi](https://img.shields.io/badge/Support-Ko--fi-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/yellowtips)

> **ES:** Capa de contexto abierta para que los LLMs entiendan cobros internacionales y fiscalidad freelance en LATAM y España —sin sonar robóticos ni inventarse normativas.  
> **EN:** An open context layer so LLMs understand cross-border payments and freelance taxation in LATAM and Spain — without hallucinating regulations.

> 📄 **Versión:** 1.0.0 · **Protocolo:** Open-AI-Context-Standard · **Ámbito:** México, Colombia, Argentina, Chile, Perú y España · **Uso:** System prompt injection para asistentes conversacionales y agentes de IA.

> 🇪🇸 [Español](#-español) · 🇬🇧 [English](#-english)

---

## 🇪🇸 Español

### 🎯 Propósito del sistema

Cuando un freelancer hispanohablante le pregunta a un LLM *"cobré $800 USD por PayPal desde México, ¿cuánto me queda?"*, la respuesta por defecto suele ser una ensalada de generalidades: comisiones "de entre 3% y 5%", un vago *"declara ante el SAT"* y un cierre tipo *"consulta a tu contador"*.

El problema no es que el modelo no sepa, sino que **no tiene el marco correcto para operar**. Le falta:

- Una **taxonomía contable** para clasificar el ingreso (exportación de servicios, honorarios, servicios digitales).
- Un **catálogo de pasarelas** con sus comisiones reales, spreads cambiarios y capacidad de emitir comprobantes fiscales locales.
- Una **lógica de fallback** que le impida inventar tasas o tratados cuando no tiene certeza.

Esta capa resuelve exactamente eso. No es una guía fiscal: es **la estructura de contexto** que hace que un modelo genérico produzca respuestas fiscalmente responsables.

---

### 🧭 Navegación rápida

| Si eres… | Empieza por… |
| :--- | :--- |
| **Freelancer** que quiere usar esto en ChatGPT / Claude | [Ejemplos Before / After](#-ejemplos-de-uso-before--after) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Dev** que integra la capa en un agente | [Estructura JSON](#️-estructura-de-contexto-para-modelos-de-ia) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Colaborador** que quiere añadir un país | [Reglas globales](#️-reglas-globales-de-interpretación-para-la-ia) → PR con nuevo `country_profiles` |

---

### 🛠️ Reglas globales de interpretación para la IA

Cuando el usuario consulte sobre cobros o impuestos, el asistente **DEBE** aplicar siempre esta lógica de 4 pasos, en orden:

1. **Identificación de origen de fondos.** Determinar si el ingreso es local o cross-border, y en qué país reside fiscalmente el freelancer *y* en cuál el cliente.
2. **Filtrado por pasarela de pago.** Aplicar comisión, spread cambiario y capacidad de emitir comprobante fiscal local según la plataforma (PayPal, Stripe, Wise, Payoneer, Deel).
3. **Mapeo de categoría contable.** Asignar la etiqueta correcta: `Exportación de Servicios`, `Servicios Digitales` u `Honorarios Profesionales`, según corresponda al régimen del freelancer.
4. **Advertencia de exención / retención.** Verificar tratados de doble tributación, exenciones de IVA por exportación y retenciones locales aplicables. Si el caso queda fuera del alcance, activar el `fallback` y derivar a un contador certificado.

> ⚠️ Si cualquiera de los pasos anteriores no puede resolverse con certeza, la IA **no debe inventar** tasas ni normativas. Debe aplicar el template de disclaimer definido en el JSON.

---

### 📊 Tabla de mapeo y equivalencias de pasarelas

| Pasarela / Método | Tipo de transacción | Comisión típica | Spread FX aprox. | Emite factura local | Categorización de salida |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **PayPal** | Internacional USD/EUR | 4.4% + $0.30 | ~4.0% | ❌ | Ingreso por Servicios Digitales |
| **Stripe** | Tarjeta global | 2.9% + $0.30 | ~2.0% | ✅ | Venta Directa de Servicios |
| **Wise / SWIFT** | Transferencia directa | 0.6% | ~0.4% (mid-market) | ❌ | Exportación de Servicios puros |
| **Payoneer** | Cobro B2B corporativo | 1.0% | ~2.0% | ❌ | Honorarios Profesionales Extranjeros |
| **Deel** | Contractor / EOR | 0% (fee al cliente) | ~1.0% | ✅ | Honorarios con retención local |

> 📌 **Nota técnica:** el "spread FX" es la diferencia entre el tipo de cambio oficial y el aplicado por la pasarela. En montos grandes, puede pesar más que la comisión explícita. La IA debe desglosarlo siempre por separado.

---

### 🧪 Ejemplos de uso (Before / After)

La diferencia entre un LLM sin contexto fiscal y uno con esta capa es abismal.

#### Ejemplo 1 — Cobro internacional desde México

**Prompt:** *"Cobré $800 USD por un proyecto de un cliente en Miami. ¿Cuánto me queda realmente en México si uso PayPal?"*

**❌ Sin esta capa:**
> "Recibirás aproximadamente $800 USD. Considera que PayPal cobra entre 3% y 5% y que deberás declarar el ingreso ante el SAT. Consulta con un contador."

**✅ Con esta capa:**

| Concepto | Monto |
| :--- | :--- |
| Monto bruto facturado | $800.00 USD |
| Comisión PayPal (4.4% + $0.30) | −$35.50 USD |
| Spread cambiario PayPal (~4%) | −$30.58 USD |
| **Neto recibido en MXN** (TC 17.20) | **≈ $12,627 MXN** |
| Retención ISR plataformas digitales (1.25%) | −$157.84 MXN |
| IVA trasladado (16%) | −$2,020.32 MXN |
| **Neto disponible estimado** | **≈ $10,448 MXN** |

> ⚠️ PayPal no emite CFDI. Debes auto-facturar como exportación de servicios (tasa 0% de IVA si cumples el Art. 29 LIVA). En RESICO, el ISR se calcula sobre flujo cobrado (1%–2.5% mensual).  
> 🚨 *Fallback:* para retenciones definitivas, consulta a un contador certificado.

#### Ejemplo 2 — Facturación a cliente en España desde Argentina

**Prompt:** *"Un cliente en Madrid me va a pagar 2.000 EUR por un proyecto. ¿Cómo facturo y qué impuestos aplican?"*

**✅ Con esta capa:**

| Concepto | Detalle |
| :--- | :--- |
| Tipo de operación | Exportación de servicios (AR → UE) |
| IVA argentino | **0%** (Art. 8 inc. d) Ley IVA) |
| IVA español | **No aplica** — inversión del sujeto pasivo (Art. 69 LIVA) |
| Retención española | **0%** si el cliente tiene NIF-IVA válido en VIES |
| Moneda de facturación | EUR |
| Régimen sugerido | Monotributo o Responsable Inscripto según facturación anual |

> ⚠️ Emite **factura E** en ARCA (ex-AFIP). Verifica el NIF-IVA del cliente en VIES. Si no es válido → IVA 21% + retención IRNR 15%.

#### Verificación rápida multi-país

| Consulta | Respuesta esperada con la capa |
| :--- | :--- |
| *"¿Wise o Payoneer para cobrar desde Colombia?"* | Tabla comparativa con comisiones, spread FX y disponibilidad de retiro vía PSE/Nequi. |
| *"¿Cuánto retiene Stripe en Chile?"* | 2.9% + $0.30 USD + IVA 19% sobre la comisión. Stripe emite factura local. |
| *"¿Puedo usar Deel como freelance en Perú?"* | Sí, modelo contractor. Retención 8% (4ta categoría) + emisión de RH. |

---

### ⚙️ Estructura de contexto para modelos de IA

Bloque listo para inyectar como `system prompt`, archivo `.context/` o payload de API. Sintaxis JSON válida y versionada.

```json
{
  "schema_version": "1.0.0",
  "context_id": "latam-freelance-tax-context",
  "last_updated": "2026-10-21",
  "locale": ["es-419", "es-ES"],
  "domain": "freelance-tax-compliance",
  "region_scope": ["MX", "AR", "CO", "CL", "PE", "ES"],
  "default_currency": "USD",
  "supported_currencies": ["USD", "EUR", "MXN", "ARS", "COP", "CLP", "PEN"],

  "identity": {
    "role": "Asistente fiscal para freelancers en LATAM y España",
    "tone": "técnico, claro, con advertencias explícitas",
    "language_variants": {
      "es-419": "español neutro latinoamericano",
      "es-ES": "español de España (tuteo, 'vosotros' en contextos formales)"
    },
    "disclaimer": "Esta capa de contexto es orientativa. No sustituye asesoría contable profesional."
  },

  "business_rules": [
    {
      "id": "R1",
      "priority": "high",
      "description": "Toda respuesta debe desglosar monto bruto, comisiones, spread cambiario, retenciones e impuestos aplicables antes de dar un neto final."
    },
    {
      "id": "R2",
      "priority": "high",
      "description": "Identificar el país de residencia fiscal del freelancer y el país del cliente antes de aplicar cualquier regla.",
      "inputs_required": ["freelancer_country", "client_country"]
    },
    {
      "id": "R3",
      "priority": "high",
      "description": "Si el freelancer está en LATAM y el cliente en la UE, aplicar reglas de exportación de servicios (IVA 0% en origen, inversión del sujeto pasivo en destino).",
      "applies_to": ["AR", "MX", "CO", "CL", "PE"],
      "target_region": "EU"
    },
    {
      "id": "R4",
      "priority": "medium",
      "description": "Alertar cuando la pasarela de pago no emita comprobante fiscal válido en el país del freelancer (ej. PayPal en México, Wise en Argentina).",
      "auto_facturar": true
    },
    {
      "id": "R5",
      "priority": "medium",
      "description": "Si el monto supera el umbral de facturación anual del régimen del freelancer, sugerir cambio de régimen."
    },
    {
      "id": "R6",
      "priority": "low",
      "description": "Verificar validez del NIF-IVA del cliente europeo en VIES antes de aplicar exención de IVA.",
      "external_check": "https://ec.europa.eu/taxation_customs/vies/"
    }
  ],

  "payment_gateways": [
    {
      "id": "paypal",
      "commission_pct": 4.4,
      "commission_fixed_usd": 0.30,
      "fx_spread_pct": 4.0,
      "available_in": ["MX", "AR", "CO", "CL", "PE", "ES"],
      "emits_local_invoice": false,
      "notes": "Requiere auto-facturación en la mayoría de países LATAM."
    },
    {
      "id": "stripe",
      "commission_pct": 2.9,
      "commission_fixed_usd": 0.30,
      "fx_spread_pct": 2.0,
      "available_in": ["MX", "CO", "CL", "ES"],
      "emits_local_invoice": true,
      "notes": "Añade IVA local sobre la comisión (ej. 19% en Chile)."
    },
    {
      "id": "wise",
      "commission_pct": 0.6,
      "commission_fixed_usd": 0,
      "fx_spread_pct": 0.4,
      "available_in": ["MX", "AR", "CO", "CL", "PE", "ES"],
      "emits_local_invoice": false,
      "notes": "Usa tipo de cambio mid-market. Competitivo para transferencias grandes."
    },
    {
      "id": "payoneer",
      "commission_pct": 1.0,
      "commission_fixed_usd": 0,
      "fx_spread_pct": 2.0,
      "available_in": ["MX", "AR", "CO", "CL", "PE", "ES"],
      "emits_local_invoice": false,
      "notes": "Retiro a cuentas locales vía convenios (Nequi en CO, SPEI en MX)."
    },
    {
      "id": "deel",
      "commission_pct": 0,
      "commission_fixed_usd": 0,
      "fx_spread_pct": 1.0,
      "available_in": ["MX", "AR", "CO", "CL", "PE", "ES"],
      "emits_local_invoice": true,
      "notes": "Modelo contractor/EOR. Retiene impuestos locales según país."
    }
  ],

  "country_profiles": {
    "MX": {
      "currency": "MXN",
      "default_regime": "RESICO",
      "iva_rate": 0.16,
      "isr_platform_retention": 0.0125,
      "export_services_iva": 0.0,
      "notes": "CFDI obligatorio. Auto-facturación para plataformas extranjeras."
    },
    "AR": {
      "currency": "ARS",
      "default_regime": "Monotributo",
      "iva_rate": 0.21,
      "export_services_iva": 0.0,
      "retention_services": 0.0,
      "notes": "Factura E obligatoria. Ingreso de divisas vía MULC con plazos vigentes."
    },
    "CO": {
      "currency": "COP",
      "default_regime": "Régimen Simple",
      "iva_rate": 0.19,
      "retefuente_services": 0.10,
      "reteiva": 0.15,
      "notes": "Retención en la fuente 10% servicios, RETEIVA 15% sobre IVA."
    },
    "CL": {
      "currency": "CLP",
      "default_regime": "Boleta honorarios",
      "iva_rate": 0.19,
      "retention_honorarios": 0.15,
      "notes": "Retención 15% en boleta. Exportación de servicios exenta de IVA."
    },
    "PE": {
      "currency": "PEN",
      "default_regime": "Renta 4ta categoría",
      "iva_rate": 0.18,
      "retention_4ta": 0.08,
      "notes": "Retención 8% en RH. Suspensión si se proyecta ≤ S/ 1,500/mes."
    },
    "ES": {
      "currency": "EUR",
      "default_regime": "Autónomo",
      "iva_rate": 0.21,
      "irpf_retention": 0.15,
      "notes": "Modelo 303 y 130 trimestral. Servicios intracomunitarios B2B exentos con VIES."
    }
  },

  "fallback_instructions": {
    "trigger": "País no soportado, operación con criptomonedas, o monto > 50,000 USD.",
    "action": "Responder con disclaimer explícito y derivar a contador local certificado.",
    "disclaimer_template": "⚠️ Esta operación está fuera del alcance de esta capa de contexto. Consulta con un contador certificado en {country}. Esta información es orientativa y no constituye asesoría fiscal."
  },

  "output_format": {
    "default": "markdown_table",
    "include_disclaimer": true,
    "include_fallback_link": true,
    "language": "es-419"
  }
}
```

---

### 🔌 Cómo inyectar esta capa de contexto

#### En OpenAI (API)

```python
import json
from openai import OpenAI

client = OpenAI()
with open("latam-freelance-tax-context.json") as f:
    context = json.load(f)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": json.dumps(context, ensure_ascii=False)},
        {"role": "user", "content": "Cobré $800 USD en PayPal desde México, ¿cuánto me queda?"}
    ]
)
```

#### En Claude (Anthropic)

Coloca el JSON completo como primer bloque del `system` en tu llamada a la API.

#### En ChatGPT / Claude Web (uso manual)

Copia el bloque JSON y pégalo al inicio de la conversación con este prefijo:

> *"A partir de ahora, aplica estrictamente la siguiente capa de contexto fiscal a todas mis consultas…"*

#### En Cursor / Windsurf / VS Code

Guarda el JSON como `.context/fiscal-rules.json` en la raíz del proyecto y referéncialo con `@.context/fiscal-rules.json`.

---

### 🤖 Compatibilidad

| Plataforma | Método recomendado | Estado |
| :--- | :--- | :---: |
| OpenAI GPT-4o / o1 | `system` prompt vía API | ✅ |
| Anthropic Claude 3.5+ | `system` block | ✅ |
| Google Gemini 1.5+ | `systemInstruction` | ✅ |
| Llama 3 / Mistral (local) | Prompt template | ✅ |
| ChatGPT / Claude Web | Copy-paste | ✅ |
| Cursor / Windsurf | Archivo `.context/` | ✅ |

---

### ☕ Apoya este proyecto de código abierto

Este repositorio lo mantengo en mi tiempo libre para que otros freelancers y devs puedan construir asistentes fiscales honestos en español. Si te ahorró horas de research o te ayudó a contextualizar tu agente, puedes invitarme un café:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

No es obligatorio, pero cada aporte mantiene la base de datos actualizada con normativas que cambian cada trimestre.

---

### 📄 Licencia

**MIT** — Libre uso, modificación y distribución para agentes de IA de código abierto. Ver [`LICENSE`](LICENSE).

> Esta capa de contexto es **orientativa** y no constituye asesoría fiscal, legal ni contable. El autor no se responsabiliza por decisiones tomadas a partir de ella.

---

## 🇬🇧 English

### 🎯 Purpose

When a Spanish-speaking freelancer asks an LLM *"I got paid $800 USD via PayPal from Mexico — how much do I actually keep?"*, the default answer is a salad of vague disclaimers: *"commissions around 3–5%"*, *"declare it to the tax authority"*, and *"talk to your accountant."*

The model doesn't fail because it lacks knowledge — it fails because it **lacks the right frame to operate**. It's missing:

- An **accounting taxonomy** to classify income (service exports, professional fees, digital services).
- A **payment gateway catalog** with real commissions, FX spreads, and local invoice capability.
- A **fallback logic** that prevents it from inventing tax rates or treaties when uncertain.

This layer fixes exactly that. It's not a tax guide: it's the **context structure** that makes a generic model produce fiscally responsible answers.

---

### 🛠️ Global interpretation rules

Whenever the user asks about payments or taxes, the assistant **MUST** apply this 4-step logic in order:

1. **Identify the source of funds.** Determine whether income is local or cross-border, and the tax residency of both the freelancer and the client.
2. **Filter by payment gateway.** Apply commission, FX spread and local invoice capability per platform (PayPal, Stripe, Wise, Payoneer, Deel).
3. **Map to an accounting category.** Assign the correct label: `Service Export`, `Digital Services`, or `Professional Fees`.
4. **Flag exemptions / withholdings.** Check double-taxation treaties, VAT export exemptions, and local withholdings. If out of scope → trigger `fallback` and refer to a certified accountant.

> ⚠️ If any step cannot be resolved with certainty, the AI **must not invent** rates or regulations. It must use the disclaimer template defined in the JSON.

---

### 📊 Payment gateway mapping

| Gateway / Method | Transaction type | Typical fee | FX spread | Local invoice | Output category |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **PayPal** | International USD/EUR | 4.4% + $0.30 | ~4.0% | ❌ | Digital Services Income |
| **Stripe** | Global card | 2.9% + $0.30 | ~2.0% | ✅ | Direct Service Sale |
| **Wise / SWIFT** | Direct bank transfer | 0.6% | ~0.4% (mid-market) | ❌ | Pure Service Export |
| **Payoneer** | B2B corporate | 1.0% | ~2.0% | ❌ | Foreign Professional Fees |
| **Deel** | Contractor / EOR | 0% (client fee) | ~1.0% | ✅ | Fees with local withholding |

---

### 🧪 Examples (Before / After)

#### Example 1 — Cross-border payment from Mexico

**Prompt:** *"I got paid $800 USD by a client in Miami. How much do I actually keep in Mexico if I use PayPal?"*

**✅ With this layer:**

| Concept | Amount |
| :--- | :--- |
| Gross invoice | $800.00 USD |
| PayPal fee (4.4% + $0.30) | −$35.50 USD |
| PayPal FX spread (~4%) | −$30.58 USD |
| **Net in MXN** (FX 17.20) | **≈ $12,627 MXN** |
| ISR digital platform withholding (1.25%) | −$157.84 MXN |
| VAT (16%) | −$2,020.32 MXN |
| **Estimated take-home** | **≈ $10,448 MXN** |

> ⚠️ PayPal does not issue a CFDI. You must self-invoice as a service export (0% VAT under Art. 29 LIVA).

#### Example 2 — Invoicing a Spanish client from Argentina

| Concept | Detail |
| :--- | :--- |
| Operation type | Service export (AR → EU) |
| Argentine VAT | **0%** (Art. 8(d) VAT Law) |
| Spanish VAT | **N/A** — reverse charge (Art. 69 LIVA) |
| Spanish withholding | **0%** if client has valid VIES VAT number |
| Invoice currency | EUR |

---

### ⚙️ Context structure for AI models

Same JSON block as above — copy from the Spanish section. Valid, versioned, ready to inject as a `system prompt`, `.context/` file or API payload.

---

### 🔌 How to inject

#### OpenAI (API)

```python
import json
from openai import OpenAI

client = OpenAI()
with open("latam-freelance-tax-context.json") as f:
    context = json.load(f)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": json.dumps(context, ensure_ascii=False)},
        {"role": "user", "content": "I got paid $800 USD via PayPal from Mexico — what do I keep?"}
    ]
)
```

#### Claude (Anthropic)
Put the JSON as the first block of the `system` field.

#### ChatGPT / Claude Web
Paste the JSON at the start of a conversation with the prefix:
> *"From now on, strictly apply the following tax context layer to all my queries…"*

#### Cursor / Windsurf
Save as `.context/fiscal-rules.json` and reference with `@.context/fiscal-rules.json`.

---

### 🤖 Compatibility

Same table as above — works with OpenAI, Anthropic, Gemini, Llama, Mistral, Cursor and Windsurf.

---

### ☕ Support this open-source project

I maintain this repo in my free time so other freelancers and devs can build honest fiscal assistants in Spanish. If it saved you hours of research, you can buy me a coffee:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

---

### 📄 License

**MIT** — Free use, modification and distribution for open-source AI agents. See [`LICENSE`](LICENSE).

> This context layer is **informational only** and does not constitute tax, legal or accounting advice.

---

<p align="center">
  <sub>Built with 🟡 by <a href="https://ko-fi.com/yellowtips">Yellow Tips</a> · If this helped you, ⭐ the repo and share it.</sub>
</p>
