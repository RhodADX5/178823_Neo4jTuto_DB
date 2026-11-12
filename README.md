# Tutorial de Neo4j con Python: partidos de fútbol

**Actividad:** practicar bases de datos de grafos con Neo4j AuraDB, consultas Cypher y su integración con Python (driver oficial `neo4j`).

Este ejercicio replica el tutorial proporcionado en clase (*Neo4j Tutorial: Using And Querying Graph Databases in Python*): se crea una instancia gratuita de Neo4j AuraDB, se importa el dataset histórico de partidos de fútbol internacional (1872-2024), y se corren consultas Cypher desde Python usando el driver oficial.

Matrícula: **178823**

## 1. Preparación

1. Crea una instancia gratuita de **Neo4j AuraDB** en <https://console.neo4j.io/> (botón "Get Started for Free") y guarda el **Connection URI**, el usuario y la contraseña que te da al crearla.
2. Clona tu fork y entra a esta carpeta:

   ```bash
   git clone https://github.com/TU-USUARIO/ERP-simulator.git
   cd ERP-simulator/neo4j_tutorial_178823
   ```

3. Crea un entorno virtual (o usa Conda, como sugiere el tutorial original) e instala las dependencias:

   ```bash
   python -m venv .venv
   source .venv/bin/activate   # En Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

4. Copia `.env.example` a `.env` y sustituye los valores de ejemplo por tus credenciales reales de AuraDB:

   ```dotenv
   NEO4J_URI=neo4j+s://TU_INSTANCIA.databases.neo4j.io
   NEO4J_USERNAME=neo4j
   NEO4J_PASSWORD=TU_PASSWORD
   NEO4J_DATABASE=neo4j
   ```

   **Nunca subas tu `.env` real a GitHub.** El `.gitignore` de esta carpeta ya lo excluye.

## 2. Comprobar la conexión

```bash
python conexion.py
```

Si todo está bien configurado, debe imprimir:

```
Conexión correcta con Neo4j AuraDB
```

## 3. Importar los datos de fútbol

El archivo `ingest_football_data.py` es el script original del tutorial (se descarga los tres CSV del dataset de Kaggle/GitHub de partidos internacionales, crea índices, y convierte las filas en nodos `Match`, `Team`, `Tournament`, `City`, `Country`, `Player` y sus relaciones mediante `MERGE`/`CREATE`).

```bash
python ingest_football_data.py
```

Tarda varios minutos (requiere Internet) y, al terminar, la instancia de AuraDB debe mostrar más de 64 mil nodos y 340 mil relaciones. **No lo vuelvas a correr sin necesidad**: las relaciones de goles (`SCORED_FOR`/`SCORED_IN`) usan `CREATE`, así que una segunda corrida las duplicaría.

Fuente del script: <https://github.com/BexTuychiev/medium_stories/blob/master/2024/9_september/3_neo4j_python/ingest_football_data.py>

## 4. Ejecutar las consultas Cypher desde Python

```bash
python consultas.py
```

`consultas.py` reproduce los ejemplos del tutorial:

- `MATCH` + `RETURN` + `COUNT` + `ORDER BY` + `LIMIT`: equipo con más victorias en la FIFA World Cup.
- La misma consulta, parametrizada (`$torneo`, `$limite`) para la UEFA Euro, demostrando por qué los parámetros son mejores que construir el string a mano (evitan inyección Cypher y permiten que Neo4j cachee el plan).
- Una transacción de lectura explícita con `session.execute_read()` para los 5 máximos goleadores históricos de la Copa Mundial.
- Una escritura con `MERGE` (no `CREATE`) para registrar un entrenador (`Lionel Scaloni` — `Argentina`, 2018-06-01) sin duplicar el nodo ni la relación si el script se corre más de una vez.

Para ver el grafo de forma visual, entra a tu instancia en AuraDB, abre la pestaña **Query** y prueba, por ejemplo:

```cypher
MATCH (t:Team)-[:WON]->(m:Match)-[:PART_OF]->(:Tournament {name: 'FIFA World Cup'})
RETURN t, m
LIMIT 20;
```

y cambia a la vista de grafo para ver nodos y relaciones.

## 5. Evidencias de la entrega

- [ ] Consola mostrando `Conexión correcta con Neo4j AuraDB` (salida de `conexion.py`).
- [ ] Proceso de `ingest_football_data.py` finalizado (`Data ingestion completed!`) y el panel de AuraDB con >64k nodos / >340k relaciones.
- [ ] Resultado de al menos una consulta Cypher visualizada en AuraDB (vista de grafo).
- [ ] Salida de `consultas.py` en terminal.
- [ ] Este pull request abierto hacia `jesusrevilla/ERP-simulator`, rama `development`.

**Seguridad:** oculta/difumina credenciales, el URI de conexión y contraseñas en cualquier captura. Nunca subas el archivo `.env` real.

## 6. Cómo se hizo el pull request

- **Repositorio base (destino):** `jesusrevilla/ERP-simulator`
- **Rama base:** `development`
- **Repositorio head:** fork propio de `ERP-simulator`
- **Rama de comparación:** `features/178823`

Pasos seguidos:

```bash
# 1) Fork del repositorio desde GitHub (botón "Fork" en jesusrevilla/ERP-simulator)

# 2) Clonar el propio fork
git clone https://github.com/TU-USUARIO/ERP-simulator.git
cd ERP-simulator

# 3) Agregar el repo original como remoto "upstream" y actualizar development
git remote add upstream https://github.com/jesusrevilla/ERP-simulator.git
git fetch upstream
git checkout development
git merge upstream/development

# 4) Crear la rama de la actividad
git checkout -b features/178823

# 5) Copiar esta carpeta (neo4j_tutorial_178823/) a la raíz del repo clonado,
#    agregar y subir los cambios
git add neo4j_tutorial_178823
git commit -m "Actividad Neo4j: tutorial de grafos con datos de fútbol (178823)"
git push origin features/178823

# 6) Abrir el pull request en GitHub:
#    base repository: jesusrevilla/ERP-simulator   base branch: development
#    head repository: TU-USUARIO/ERP-simulator     compare branch: features/178823
```

## Fuentes

- Tutorial de Neo4j compartido en la actividad (*Neo4j Tutorial: Using And Querying Graph Databases in Python*).
- Script de importación original: <https://github.com/BexTuychiev/medium_stories/blob/master/2024/9_september/3_neo4j_python/ingest_football_data.py>
- Dataset: <https://github.com/martj42/international_results>
- Documentación oficial: <https://neo4j.com/docs/aura/> · <https://neo4j.com/docs/python-manual/current/>
