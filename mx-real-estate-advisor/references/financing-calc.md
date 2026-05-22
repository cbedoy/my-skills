# Financiamiento Hipotecario — Cálculos y Comparativa

> ⚠️ Tasas bancarias cambian frecuentemente. Siempre verificar con web_search antes de presentar tabla comparativa.

---

## Fórmula de amortización (cuota fija mensual)

```
M = P × [r(1+r)^n] / [(1+r)^n - 1]

Donde:
  M = mensualidad fija
  P = monto del préstamo (principal)
  r = tasa de interés mensual = tasa anual / 12
  n = número de pagos (meses) = años × 12
```

### Ejemplo

```
Préstamo: $1,500,000 MXN
Tasa anual: 10.5%
Plazo: 20 años (240 meses)

r = 0.105 / 12 = 0.00875
n = 240

M = 1,500,000 × [0.00875 × (1.00875)^240] / [(1.00875)^240 - 1]
  = 1,500,000 × [0.00875 × 7.926] / [7.926 - 1]
  = 1,500,000 × 0.06935 / 6.926
  ≈ $15,022 MXN/mes

Total pagado = 15,022 × 240 = $3,605,280 MXN
Total intereses = 3,605,280 - 1,500,000 = $2,105,280 MXN
```

---

## Tasas hipotecarias bancarias vigentes (referencia 2024-2025)

> Verificar con web_search "tasas hipotecarias México 2025" antes de usar.

| Banco | Tasa desde | Plazo máx | CAT aprox |
|---|---|---|---|
| BBVA | 9.90% | 20 años | 11.5% |
| Banorte | 9.75% | 20 años | 11.2% |
| Scotiabank | 10.30% | 20 años | 12.1% |
| HSBC | 10.50% | 20 años | 12.3% |
| Inbursa | 9.50% | 20 años | 10.9% |
| Santander | 10.20% | 20 años | 11.8% |

La tasa real depende del perfil del solicitante (buró, ingreso, enganche).  
Enganche mínimo bancario: generalmente 10–20% del valor del inmueble.

---

## Tabla comparativa de esquemas (template)

Calcular para el perfil del usuario y presentar así:

| Esquema | Precio inmueble | Enganche | Monto financiado | Tasa | Plazo | Mensualidad | Total intereses | Costo total |
|---|---|---|---|---|---|---|---|---|
| Solo Infonavit | | | | | | | | |
| Cofinavit (50/50) | | | | | | | | |
| Bancario 80/20 | | | | | | | | |
| Bancario + Apoyo Infonavit | | | | | | | | |

---

## Regla del 30% (capacidad de pago)

```
Mensualidad máxima sana = Ingreso neto × 0.30
Monto máximo financiable ≈ Mensualidad sana / r × [1 - (1+r)^(-n)]
```

Si la mensualidad calculada supera el 30%, bajar el precio objetivo o ampliar el plazo.

---

## Costos de cierre — estimado total

Para no llevarse sorpresas, calcular siempre un **colchón del 8-10%** del valor del inmueble:

```
Costo de cierre estimado = Precio × 0.09 (promedio)

Incluye:
- Escrituración + notaría: 4-7%
- Avalúo: ~$6,000 MXN fijos
- Apertura de crédito bancario: 1-2%
- Seguros primer año: ~0.5%
- Gastos varios (gestoría, copias, traslados): ~0.5%
```

Este monto debe salir del **ahorro líquido**, no del crédito hipotecario.

---

## Costo real de oportunidad

Antes de recomendar esquema, calcular también:

```
Si el usuario invierte el ahorro líquido en lugar de usarlo como enganche:

Rendimiento alternativo = Ahorro × tasa CETES/fondo (verificar tasa vigente)
Costo de mayor mensualidad = mensualidad sin enganche - mensualidad con enganche

Si rendimiento alternativo > ahorro en mensualidad → considerar no usar todo el ahorro como enganche
```

---

## Plusvalía como factor de retorno

```
Retorno sobre capital (ROC) simple a N años:
  Valor futuro = Precio compra × (1 + tasa plusvalía anual)^N
  ROC = (Valor futuro - Precio compra - Total intereses) / Capital propio invertido

Para zonas en crecimiento real en México: asumir 4-8% anual de plusvalía.
Para zonas maduras o estancadas: asumir 2-4%.
Siempre especificar el supuesto usado.
```
