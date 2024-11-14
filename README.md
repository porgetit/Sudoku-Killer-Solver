# Solucionador de Sudoku Killer Usando CSP y Algoritmo Genético

Este proyecto implementa un solucionador para Sudoku Killer, una variante del Sudoku que comienza con un tablero vacío y depende de restricciones de suma en regiones específicas. El solucionador utiliza técnicas de Problemas de Satisfacción de Restricciones (CSP) para reducir el espacio de búsqueda y emplea un algoritmo genético, implementado con la biblioteca DEAP, para buscar soluciones viables.

## Enfoque

1. **Reducción de Dominios mediante CSP**  
   Utilizando técnicas de CSP, comenzamos reduciendo los valores posibles para cada celda en función de las sumas dadas en cada región. Por ejemplo, si una región tiene una suma objetivo de 8, los valores posibles para las celdas de esa región se restringen a combinaciones que sumen 8. Esta reducción inicial de dominio limita significativamente el espacio de búsqueda.

2. **Manejo de Pares, Tríos y Restricciones de Grupo**  
   Estrategias de restricciones adicionales, como la regla de pares o tríos, reducen aún más las posibilidades de cada celda al identificar valores compartidos en grupos de celdas. Estas estrategias optimizan los dominios asegurando que los valores sigan siendo consistentes con las restricciones del Sudoku.

3. **Algoritmo Genético para la Búsqueda de Soluciones**  
   Una vez que el CSP ha limitado los valores posibles, aplicamos un algoritmo genético para realizar la búsqueda dentro del espacio factible. El algoritmo utiliza 81 genes por individuo, cada uno representando una celda en el tablero de Sudoku. Inicializamos cada gen en función de su dominio reducido usando DEAP.

4. **Función Objetivo y Evaluación de Restricciones**  
   La función objetivo evalúa cada solución candidata calculando cuántas restricciones se están violando. Esto incluye:
   - **Violaciones de Suma**: No cumplir con la suma de una región se considera una violación menor.
   - **Violaciones de las Reglas de Sudoku**: Las violaciones de las reglas básicas del Sudoku (valores únicos en filas, columnas y bloques) se consideran más graves.  
   Este sistema de ponderación ayuda a priorizar soluciones que cumplan con las restricciones fundamentales del Sudoku antes de optimizar las sumas de las regiones.

5. **Terminación Temprana al Encontrar una Solución**  
   Para mejorar la eficiencia, el algoritmo se detiene tan pronto como encuentra una solución válida, evitando búsquedas adicionales una vez que se logra una configuración factible.

## Asignación de Restricciones de Dominio para Cada Gen

Cada gen representa una celda en el tablero de Sudoku y se inicializa con un dominio específico en función de sus restricciones. El siguiente código ilustra cómo DEAP puede configurarse para establecer cada gen individualmente.

```python
from deap import base, creator, tools
import random

# Definir el problema de maximización
creator.create("FitnessMax", base.Fitness, weights=(1.0,))
creator.create("Individual", list, fitness=creator.FitnessMax)

# Configurar la caja de herramientas y dominios para cada gen
toolbox = base.Toolbox()

# Definir inicializadores específicos para cada gen en función de restricciones de ejemplo
def gene_a():
    return random.choice([1, 2, 3, 4])

def gene_b():
    return random.choice([1, 2, 3, 5, 6, 7, 8])

# Registrar la creación de genes y del individuo
toolbox.register("gene_a", gene_a)
toolbox.register("gene_b", gene_b)
toolbox.register("individual", tools.initCycle, creator.Individual, (toolbox.gene_a, toolbox.gene_b), n=1)

# Crear la población
toolbox.register("population", tools.initRepeat, list, toolbox.individual)

# Función de evaluación de ejemplo
def evalCustom(individual):
    return sum(individual),  # Evaluación de ejemplo

toolbox.register("evaluate", evalCustom)
toolbox.register("mate", tools.cxTwoPoint)
toolbox.register("mutate", tools.mutFlipBit, indpb=0.5)
toolbox.register("select", tools.selTournament, tournsize=3)

# Generar y evaluar población
population = toolbox.population(n=10)
for ind in population:
    print("Individuo:", ind)
    print("Fitness:", evalCustom(ind)[0])
```

En el código, los genes individuales `A` y `B` tienen dominios separados. Este patrón se puede extender para configuraciones más complejas, adaptando cada gen a las restricciones específicas de su celda.

## Direcciones Futuras

- **Refinamiento de Restricciones**: Ajustar los pesos de la función objetivo para violaciones de suma frente a violaciones de reglas podría mejorar la calidad de la solución.
- **Mutación Adaptativa**: Ajustar las probabilidades de mutación para mejorar la velocidad de convergencia mientras se preserva la diversidad de soluciones.
- **Evaluación en Paralelo**: Usar computación paralela para acelerar la evaluación, especialmente útil para grandes poblaciones.

---

Este enfoque estructurado combina técnicas de CSP con algoritmos genéticos, buscando desarrollar un solucionador de Sudoku Killer eficiente y adaptable.
