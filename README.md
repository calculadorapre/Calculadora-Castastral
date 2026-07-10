README.md:

# Verificador de Impuesto Predial — Colombia

Chatbot que verifica si el aumento del impuesto predial respeta los topes legales en Colombia, según la **Ley 44 de 1990** y la **Ley 1995 de 2019**.

## Funcionamiento

1. El usuario responde 17 preguntas sobre su predio y situación tributaria.
2. El chatbot aplica las 5 reglas de decisión:
   - **Regla 1:** Excepciones legales (sin límite de incremento).
   - **Regla 2:** Estrato 1–2 con avalúo ≤ 135 SMMLV (tope = IPC).
   - **Regla 3:** Predio actualizado catastralmente (tope = IPC + 8 puntos).
   - **Regla 4:** Sin actualización catastral (tope = 50%).
   - **Regla 5:** Primer cobro tras actualización (tope = 100%).
3. Muestra si el cobro respeta o supera el límite legal.

## Requisitos

- Python 3.10 o superior

## Ejecución

```bash
python chatbot.py
