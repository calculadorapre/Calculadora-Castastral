   # ============================================================
# Chatbot: Verificador de límites del Impuesto Predial
# Basado en: Preguntas_y_Reglas_Impuesto_Predial.docx
# Ejecutar en Google Colab o en consola con Python 3.10+
# ============================================================


def preguntar_texto(mensaje):
    """Pide un dato de texto libre (no vacío)."""
    while True:
        respuesta = input(mensaje + "\n> ").strip()
        if respuesta:
            return respuesta
        print("Por favor escribe una respuesta.\n")


def preguntar_numero(mensaje, minimo=None, maximo=None, permitir_decimales=True):
    """Pide un número, validando tipo, rango y que no sea negativo."""
    while True:
        respuesta = input(mensaje + "\n> ").strip().replace(",", "")
        try:
            valor = float(respuesta) if permitir_decimales else int(respuesta)
            if minimo is not None and valor < minimo:
                print(f"El valor no puede ser menor que {minimo}.\n")
                continue
            if maximo is not None and valor > maximo:
                print(f"El valor no puede ser mayor que {maximo}.\n")
                continue
            return valor
        except ValueError:
            print("Escribe solo un número, por ejemplo 950000 o 5.2\n")


def preguntar_si_no(mensaje):
    """Pide una respuesta Sí/No y la devuelve como booleano."""
    afirmativas = {"si", "sí", "s", "yes", "y"}
    negativas = {"no", "n"}
    while True:
        respuesta = input(mensaje + " (sí/no)\n> ").strip().lower()
        if respuesta in afirmativas:
            return True
        if respuesta in negativas:
            return False
        print("Responde 'sí' o 'no'.\n")


def saludo_inicial():
    """Muestra el banner de bienvenida y explica el propósito del chatbot."""
    print("=" * 64)
    print(" VERIFICADOR DEL IMPUESTO PREDIAL — LEY 44 DE 1990 Y LEY 1995 DE 2019")
    print("=" * 64)
    print(
        "\n¡Hola! Soy tu asistente para revisar si el aumento de tu impuesto\n"
        "predial respeta los topes legales en Colombia.\n\n"
        "Te voy a hacer una serie de preguntas sencillas. Al final te diré\n"
        "cuál es el tope máximo que la ley permite y si el cobro que\n"
        "recibiste parece ajustarse a la norma.\n\n"
        "Esta herramienta es informativa: la decisión final siempre la\n"
        "tiene la Secretaría de Hacienda de tu municipio.\n"
    )


def recopilar_datos():
    """Hace las 17 preguntas del documento, en el orden indicado, y arma el diccionario de variables."""
    datos = {}

    datos["anio_impuesto"] = int(preguntar_numero(
        "1. ¿En qué año corresponde el impuesto que desea revisar?",
        minimo=1900, maximo=2100, permitir_decimales=False
    ))

    datos["impuesto_anterior"] = preguntar_numero(
        "2. ¿Cuánto pagó de impuesto predial el año anterior? (en pesos)",
        minimo=0
    )

    datos["impuesto_actual"] = preguntar_numero(
        "3. ¿Cuánto le están cobrando este año? (en pesos)",
        minimo=0
    )

    datos["catastro_actualizado"] = preguntar_si_no(
        "4. ¿Su predio fue actualizado catastralmente antes de este cobro?"
    )

    if datos["catastro_actualizado"]:
        datos["primer_cobro_actualizacion"] = preguntar_si_no(
            "5. ¿Este es el primer impuesto calculado con ese nuevo avalúo?"
        )
    else:
        datos["primer_cobro_actualizacion"] = False
        print("  (Se asume 'No' para pregunta 5)\n")

    datos["es_formacion_catastral"] = preguntar_si_no(
        "6. ¿Su predio fue formado por primera vez en el catastro este año?"
    )

    datos["primera_incorporacion"] = preguntar_si_no(
        "7. ¿El predio estaba por fuera del catastro y fue incorporado por primera vez?"
    )

    datos["nueva_construccion"] = preguntar_si_no(
        "8. ¿El predio era un lote y durante el último año construyó una vivienda o edificio?"
    )

    datos["lote_sin_edificar"] = preguntar_si_no(
        "9. ¿El predio es urbanizable no urbanizado o urbanizado no edificado?"
    )

    datos["usa_autoavaluo"] = preguntar_si_no(
        "10. ¿El impuesto se calcula con autoavalúo y no con avalúo catastral?"
    )

    datos["cambio_destino"] = preguntar_si_no(
        "11. ¿El destino económico del predio cambió?"
    )

    datos["cambio_area"] = preguntar_si_no(
        "12. ¿Cambió el área del terreno o el área construida?"
    )

    datos["es_rural"] = preguntar_si_no(
        "13. ¿Su predio es rural?"
    )

    if datos["es_rural"]:
        datos["hectareas"] = preguntar_numero(
            "14. ¿Cuántas hectáreas tiene el predio?",
            minimo=0
        )
    else:
        datos["hectareas"] = 0

    datos["estrato"] = int(preguntar_numero(
        "15. ¿A qué estrato pertenece su vivienda? (escriba 0 si no aplica)",
        minimo=0, maximo=6, permitir_decimales=False
    ))

    datos["avaluo_menor_135_smmlv"] = preguntar_si_no(
        "16. ¿El avalúo catastral es menor o igual a 135 salarios mínimos (SMMLV)?"
    )

    datos["ipc"] = preguntar_numero(
        "17. ¿Cuál fue el IPC (inflación) utilizado para ese año? (ej: 5.2 para 5.2%)",
        minimo=0, maximo=100
    )

    return datos


def confirmar_datos(datos):
    """Muestra un resumen de las respuestas y pide confirmación."""
    print("\n" + "-" * 64)
    print(" RESUMEN DE TUS RESPUESTAS")
    print("-" * 64)
    print(f"  Año:                    {datos['anio_impuesto']}")
    print(f"  Impuesto anterior:      ${datos['impuesto_anterior']:,.0f}")
    print(f"  Impuesto este año:      ${datos['impuesto_actual']:,.0f}")
    print(f"  IPC:                    {datos['ipc']}%")
    print(f"  Estrato:                {datos['estrato']}")
    print(f"  Rural:                  {'Sí' if datos['es_rural'] else 'No'}")
    if datos["es_rural"]:
        print(f"  Hectáreas:              {datos['hectareas']}")
    print(f"  Actualizado catastral:  {'Sí' if datos['catastro_actualizado'] else 'No'}")
    print(f"  Primer cobro post-act.: {'Sí' if datos['primer_cobro_actualizacion'] else 'No'}")
    print(f"  Formación catastral:    {'Sí' if datos['es_formacion_catastral'] else 'No'}")
    print(f"  Primera incorporación:  {'Sí' if datos['primera_incorporacion'] else 'No'}")
    print(f"  Nueva construcción:     {'Sí' if datos['nueva_construccion'] else 'No'}")
    print(f"  Lote sin edificar:      {'Sí' if datos['lote_sin_edificar'] else 'No'}")
    print(f"  Autoavalúo:             {'Sí' if datos['usa_autoavaluo'] else 'No'}")
    print(f"  Cambio destino:         {'Sí' if datos['cambio_destino'] else 'No'}")
    print(f"  Cambio área:            {'Sí' if datos['cambio_area'] else 'No'}")
    print(f"  Avalúo ≤135 SMMLV:      {'Sí' if datos['avaluo_menor_135_smmlv'] else 'No'}")
    print("-" * 64)
    return preguntar_si_no("¿Los datos son correctos?")


def evaluar_predial(datos):
    """
    Aplica la lógica de decisión del documento:
    Regla 1: excepciones legales -> sin límite de incremento.
    Regla 2: estrato 1-2 y avalúo <=135 SMMLV -> tope = anterior * (1 + IPC).
    Regla 5: primer cobro tras actualización catastral -> tope = 100% (el doble).
    Regla 3: catastro actualizado -> tope = IPC + 8 puntos.
    Regla 4: sin actualización -> tope = 50% adicional.
    """
    rural_excede_100ha = datos["es_rural"] and datos["hectareas"] > 100

    excepciones_activas = {
        "Primera incorporación al catastro": datos["primera_incorporacion"],
        "Formación catastral inicial": datos["es_formacion_catastral"],
        "Nueva construcción sobre un lote": datos["nueva_construccion"],
        "Lote urbanizable no urbanizado o no edificado": datos["lote_sin_edificar"],
        "Uso de autoavalúo": datos["usa_autoavaluo"],
        "Cambio de destino económico": datos["cambio_destino"],
        "Cambio de área de terreno o construcción": datos["cambio_area"],
        "Predio rural mayor a 100 hectáreas": rural_excede_100ha,
    }

    activas = [nombre for nombre, aplica in excepciones_activas.items() if aplica]

    if activas:
        return {
            "regla": "Regla 1 — Excepción legal",
            "tope": None,
            "detalle": (
                "No aplica ningún límite de incremento porque se cumple al menos "
                "una excepción legal: " + "; ".join(activas) + "."
            ),
            "cumple": None,
        }

    if datos["estrato"] in (1, 2) and datos["avaluo_menor_135_smmlv"]:
        tope = datos["impuesto_anterior"] * (1 + datos["ipc"] / 100)
        return {
            "regla": "Regla 2 — Estrato 1 o 2, avalúo <= 135 SMMLV",
            "tope": tope,
            "detalle": (
                f"Regla 2: vivienda de estrato {datos['estrato']} con avalúo <= 135 SMMLV. "
                f"Tope máximo = impuesto anterior x (1 + IPC) = "
                f"${datos['impuesto_anterior']:,.0f} x (1 + {datos['ipc']:.1f}%)"
            ),
            "cumple": datos["impuesto_actual"] <= tope,
        }

    if datos["catastro_actualizado"] and datos["primer_cobro_actualizacion"]:
        tope = datos["impuesto_anterior"] * 2
        return {
            "regla": "Regla 5 — Primer cobro tras actualización catastral",
            "tope": tope,
            "detalle": (
                "Regla 5: es el primer cobro tras la actualización catastral. "
                "Tope máximo = el doble (100%) del impuesto del año anterior."
            ),
            "cumple": datos["impuesto_actual"] <= tope,
        }

    if datos["catastro_actualizado"]:
        incremento = datos["ipc"] + 8
        tope = datos["impuesto_anterior"] * (1 + incremento / 100)
        return {
            "regla": "Regla 3 — Predio actualizado catastralmente",
            "tope": tope,
            "detalle": (
                f"Regla 3: predio actualizado catastralmente. "
                f"Tope máximo = IPC + 8 puntos porcentuales = {incremento:.1f}%."
            ),
            "cumple": datos["impuesto_actual"] <= tope,
        }

    tope = datos["impuesto_anterior"] * 1.5
    return {
        "regla": "Regla 4 — Sin actualización catastral",
        "tope": tope,
        "detalle": "Regla 4: sin actualización catastral. El aumento no puede superar el 50%.",
        "cumple": datos["impuesto_actual"] <= tope,
    }


def mostrar_resultado(datos, resultado):
    """Imprime el resultado de la verificación con formato."""
    print("\n" + "=" * 64)
    print(" RESULTADO DE LA VERIFICACIÓN")
    print("=" * 64)
    print(f"Año revisado: {datos['anio_impuesto']}")
    print(f"Impuesto año anterior: ${datos['impuesto_anterior']:,.0f}")
    print(f"Impuesto cobrado este año: ${datos['impuesto_actual']:,.0f}")
    print(f"Regla aplicada: {resultado['regla']}")
    print(f"Detalle: {resultado['detalle']}")

    if resultado["tope"] is None:
        print("\nNo hay techo legal aplicable a este predio.")
    else:
        print(f"\nTope máximo permitido este año: ${resultado['tope']:,.0f}")
        if resultado["cumple"]:
            print("✓ El cobro recibido SÍ respeta el límite legal.")
        else:
            diferencia = datos["impuesto_actual"] - resultado["tope"]
            print("✗ El cobro recibido SUPERA el límite legal permitido.")
            print(f"  Exceso: ${diferencia:,.0f}")
            print("  Podrías presentar un recurso de reconsideración ante tu municipio.")
    print("=" * 64)


def mensaje_cierre():
    """Muestra el mensaje de despedida."""
    print("\n" + "=" * 64)
    print(" GRACIAS POR USAR EL VERIFICADOR DE IMPUESTO PREDIAL")
    print("=" * 64)
    print(
        "\nRecuerda que esta herramienta es orientativa y no sustituye\n"
        "la asesoría de un abogado tributarista o la decisión oficial\n"
        "de la Secretaría de Hacienda de tu municipio.\n\n"
        "Si detectaste un posible exceso en tu cobro, puedes:\n"
        "  1. Presentar un recurso de reconsideración (Art. 72, Estatuto Tributario).\n"
        "  2. Acudir a la Inspección de Tributos Municipales.\n"
        "  3. Consultar con un abogado especializado.\n\n"
        "¡Mucho éxito! Vuelve a usar esta herramienta cuando la necesites.\n"
    )
    print("=" * 64)


def ejecutar_chatbot():
    """Orquesta todo el flujo del chatbot."""
    saludo_inicial()

    while True:
        datos = recopilar_datos()
        if confirmar_datos(datos):
            break
        print("\nVolvamos a empezar.\n")

    resultado = evaluar_predial(datos)
    mostrar_resultado(datos, resultado)
    mensaje_cierre()

    return datos, resultado


if __name__ == "__main__":
    ejecutar_chatbot()
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
