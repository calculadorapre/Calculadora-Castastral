# Clonar el repositorio
git clone https://github.com/tu-usuario/predial-chatbot.git
cd predial-chatbot

# Ejecutar el chatbot
python3 chatbot_predial.py
#!/usr/bin/env python3
# ============================================================
# Chatbot: Verificador de límites del Impuesto Predial (Colombia)
# Basado en: Preguntas_y_Reglas_Impuesto_Predial.docx
# Aplica la Ley 44 de 1990 y la Ley 1995 de 2019
#
# Uso:
#   python3 chatbot_predial.py
# ============================================================

def preguntar_texto(mensaje):
    """Pide un dato de texto libre (no vacío)."""
    while True:
        respuesta = input(mensaje + "\n> ").strip()
        if respuesta:
            return respuesta
        print("Por favor escribe una respuesta.\n")


def preguntar_numero(mensaje, permitir_decimales=True):
    """Pide un número, validando que sea numérico y no negativo."""
    while True:
        respuesta = input(mensaje + "\n> ").strip().replace(",", "")
        try:
            valor = float(respuesta) if permitir_decimales else int(respuesta)
            if valor < 0:
                print("El valor no puede ser negativo.\n")
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
        permitir_decimales=False
    ))

    datos["impuesto_anterior"] = preguntar_numero(
        "2. ¿Cuánto pagó de impuesto predial el año anterior? (en pesos)"
    )

    datos["impuesto_actual"] = preguntar_numero(
        "3. ¿Cuánto le están cobrando este año? (en pesos)"
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
            "14. ¿Cuántas hectáreas tiene el predio?"
        )
    else:
        datos["hectareas"] = 0

    datos["estrato"] = int(preguntar_numero(
        "15. ¿A qué estrato pertenece su vivienda? (escriba 0 si no aplica)",
        permitir_decimales=False
    ))

    datos["avaluo_menor_135_smmlv"] = preguntar_si_no(
        "16. ¿El avalúo catastral es menor o igual a 135 salarios mínimos (SMMLV)?"
    )

    datos["ipc"] = preguntar_numero(
        "17. ¿Cuál fue el IPC (inflación) utilizado para ese año? (ej: 5.2 para 5.2%)"
    )

    return datos


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
        detalle = (
            f"Regla 2: vivienda de estrato {datos['estrato']} con avalúo <= 135 SMMLV. "
            f"Tope máximo = impuesto anterior x (1 + IPC) = "
            f"{datos['impuesto_anterior']:,.0f} x (1 + {datos['ipc']}%)."
        )
        regla = "Regla 2 — Estrato 1 o 2, avalúo <= 135 SMMLV"

    elif datos["catastro_actualizado"] and datos["primer_cobro_actualizacion"]:
        tope = datos["impuesto_anterior"] * 2
        detalle = (
            "Regla 5: es el primer cobro tras la actualización catastral. "
            "Tope máximo = el doble (100%) del impuesto del año anterior."
        )
        regla = "Regla 5 — Primer cobro tras actualización catastral"

    elif datos["catastro_actualizado"]:
        incremento = datos["ipc"] + 8
        tope = datos["impuesto_anterior"] * (1 + incremento / 100)
        detalle = (
            f"Regla 3: predio actualizado catastralmente. "
            f"Tope máximo = IPC + 8 puntos porcentuales = {incremento:.1f}%."
        )
        regla = "Regla 3 — Predio actualizado catastralmente"

    else:
        tope = datos["impuesto_anterior"] * 1.5
        detalle = "Regla 4: sin actualización catastral. El aumento no puede superar el 50%."
        regla = "Regla 4 — Sin actualización catastral"

    cumple = datos["impuesto_actual"] <= tope

    return {
        "regla": regla,
        "tope": tope,
        "detalle": detalle,
        "cumple": cumple,
    }


def mostrar_resultado(datos, resultado):
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
            print("El cobro recibido SÍ respeta el límite legal.")
        else:
            print("El cobro recibido SUPERA el límite legal permitido.")
            print("Podrías presentar un recurso de reconsideración ante tu municipio.")
    print("=" * 64)
    print(
        "\nNota: Este resultado es una estimación basada únicamente en las\n"
        "respuestas que diste. Los valores pueden variar según el avalúo\n"
        "y la tarifa oficial vigente. Te recomendamos verificar tus datos\n"
        "catastrales directamente en la Secretaría de Hacienda o la Oficina\n"
        "de Catastro de tu municipio antes de tomar cualquier decisión."
    )
    print("=" * 64)


def ejecutar_chatbot():
    saludo_inicial()
    datos = recopilar_datos()
    resultado = evaluar_predial(datos)
    mostrar_resultado(datos, resultado)
    return datos, resultado


# Ejecuta este archivo directamente para iniciar el chatbot:
if __name__ == "__main__":
    ejecutar_chatbot()
