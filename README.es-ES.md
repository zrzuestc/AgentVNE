# AgentVNE

AgentVNE: LLM-Aumentado Graph Reinforcement Learning para Asignación Multiagente Basada en Afinidad en Edge Agentic AI

## Descripción del Proyecto

AgentVNE es un marco de inserción de red virtual diseñado para escenarios de computación en el borde con agentes. Utiliza una arquitectura de doble capa que combina percepción semántica con aprendizaje de similitud de gráficos para abordar los desafíos de implementación de flujos de trabajo dinámicos en infraestructura heterogénea en el borde.

Todos los códigos se publicarán después de que el artículo sea aceptado.

**Ventajas Clave:**
- **Percepción Semántica**: Identifica las restricciones semánticas implícitas de los nodos de flujo de trabajo a través de LLM
- **Aprendizaje de Similitud de Gráficos**: Estrategia de pre-entrenamiento + afinación PPO para capturar con precisión las similitudes topológicas
- **Adaptación Dinámica**: Soporta llegadas de flujos de trabajo en tiempo real y cambios de recursos dinámicos
- **Mejora de Rendimiento**: Reduce la latencia de comunicación a menos del 40% de las líneas base y mejora la tasa de aceptación en un 5%-10%

## Arquitectura del Sistema

AgentVNE adopta una arquitectura de doble capa que combina percepción semántica con razonamiento topológico:

### Capa 1: Percepción Semántica basada en LLM & Resolución de Restricciones

Realiza percepción semántica y resolución de restricciones a través de modelos de lenguaje grandes, permitiendo una coincidencia inteligente de nodos y complemento de recursos.

**Funciones Principales:**
- 🔍 **Percepción Semántica**: Analiza las instrucciones de los nodos virtuales (VN) para comprender los requisitos semánticos y características funcionales
- 🎯 **Identificación de Restricciones**: Identifica automáticamente los entornos de ejecución especiales requeridos por los nodos (por ejemplo, entorno de seguridad PCI DSS, entorno de cómputo GPU, hardware de cámara, etc.)
- 🔗 **Emparejamiento Inteligente**: Empareja los nodos VN que requieren entornos de ejecución especiales con nodos apropiados de la red sustrato (SN)
- 📊 **Complemento de Recursos**: Proporciona información de restricciones a nivel semántico y datos de sesgo para decisiones de inserción posteriores

**Módulo de Implementación:** `LLM_resource_augmentation/node_optimizer/`

Este módulo utiliza LLM para analizar las instrucciones de los nodos de flujo de trabajo, determina si los nodos requieren entornos de ejecución especiales y los empareja automáticamente con nodos SN apropiados, proporcionando restricciones semánticas para las decisiones de inserción de la Capa 2.

**Ejemplo de Uso:**
```bash
cd LLM_resource_augmentation/node_optimizer
uv run run_optimizer.py
```

### Capa 2: Incrustación Profunda de Similitud de Gráficos & Optimización de Políticas

Capa de incrustación y optimización de políticas basada en redes neuronales profundas y aprendizaje por refuerzo.

**Funciones Principales:**
- 🧠 **Codificación de Gráficos**: Utiliza codificadores GCN para procesarlos por separado los gráficos de VN y SN, extrayendo características de nodos
- 🔄 **Mejora con Transformer**: Mejora las representaciones de características de nodos a través del Transformer Encoder
- 🎯 **Cálculo de Similitud**: Utiliza la Red Neuronal Tensorial (NTN) para calcular probabilidades de emparejamiento entre nodos VN y nodos SN
- 🚀 **Optimización PPO**: Afinar la red de políticas en entornos reales utilizando el algoritmo de Optimización de Políticas Próximas (PPO)

**Pipeline de Entrenamiento:**
1. **Fase de Pre-Entrenamiento**: Aprendizaje supervisado usando etiquetas NodeRank para aprender representaciones de similitud de gráficos
2. **Fase de Afinación**: Optimiza la política usando el algoritmo PPO en entornos dinámicos para maximizar la tasa de aceptación y la utilización de recursos

**Módulos de Implementación:**
- `model.py`: Modelo SimuVNE (red de políticas)
- `pretrain.py`: Script de pre-entrenamiento
- `fine_tuning.py`: Script de afinación PPO
- `env.py`: Entorno de aprendizaje por refuerzo (SimuVNEEnv)

## Características Principales

- 🎯 **Dos Etapas de Entrenamiento**: Pre-entrenamiento + afinación PPO
- 🧠 **Redes Neuronales Profundas**: Codificador GCN + Codificador Transformer
- 🔄 **Aprendizaje por Refuerzo**: Algoritmo PPO para optimización de políticas
- 📊 **Soporte Multi-Estrategia**: Métodos de línea base incluyendo codicioso, algoritmo genético, NodeRank, etc.
- 🔍 **Percepción Semántica**: Identificación y complemento de restricciones impulsado por LLM

## Inicio Rápido

### Configuración del Entorno

```bash
conda env create -f environment.yml
conda activate AgentVNE
```

### Preparación de Datos

1. Colocar archivos de topología SN en el directorio `topo/`
2. Colocar archivos de topología de Flujo de Trabajo en el directorio `Workflow_topo/`
3. Generar conjunto de datos para pre-entrenamiento:

```bash
python dataset_generate_1.py \
    --sn_topo topo/SN_topology.json \
    --workflow_topo Workflow_topo/workflow1_topo.json \
    --workflow_noderank Workflow_topo/workflow1_noderank.json \
    --output pretrain_data/pretrain_dataset.pt \
    --workflows_per_episode 10 \
    --num_episodes 50
```

### Pipeline de Entrenamiento

**1. Pre-entrenamiento**
```bash
python pretrain.py \
    --data_path pretrain_data/pretrain_dataset.pt \
    --output_dir pretrain_outputs \
    --batch_size 16 \
    --num_epochs 100 \
    --learning_rate 0.001
```

**2. Afinación**
```bash
python fine_tuning.py \
    --pretrain_model pretrain_outputs/checkpoint_latest.pt \
    --sn_topology topo/SN_topology.json \
    --workflow_types Workflow_topo/workflow1_topo.json \
    --output_dir finetuning_output \
    --num_episodes 1000 \
    --max_arrived_tasks 100
```

**3. Prueba & Evaluación**
```bash
python tester.py \
    --sn_topology topo/SN_topology.json \
    --workflow workflow1=Workflow_topo/workflow1_topo.json \
    --strategy ga --strategy greedy --strategy pretrain --strategy finetuned \
    --parameter arrival_rate=0.25,mean_lifetime=40,max_time_steps=11000,seed=42 \
    --plot
```

## Estructura del Proyecto

```
agentvne/
├── model.py                    # Modelo SimuVNE (red de políticas)
├── model__sigmoid.py           # Variante del modelo SimuVNE (con Sigmoid)
├── env.py                      # Definición del entorno (SimuVNEEnv, WorkflowGenerator)
├── pretrain.py                 # Script de pre-entrenamiento
├── fine_tuning.py              # Script de afinación PPO
├── dataset_generate_1.py       # Generación de conjunto de datos
├── tester.py                   # Script de prueba multi-estrategia
├── LLM_resource_augmentation/  # Capa 1: Percepción semántica LLM & resolución de restricciones
│   └── node_optimizer/         # Optimizador de nodos (emparejamiento VN-SN inteligente)
├── baselines/                  # Métodos de línea base
├── topo/                       # Archivos de topología SN y herramientas
├── Workflow_topo/              # Archivos de topología de Flujo de Trabajo
├── pretrain_data/              # Conjunto de datos para pre-entrenamiento
├── pretrain_outputs/           # Salidas del modelo de pre-entrenamiento
└── finetuning_output/          # Salidas del modelo de afinación
```

## Arquitectura del Modelo

El **modelo SimuVNE** consta de los siguientes componentes:

- **Codificador GCN**: Codifica los gráficos de VN y SN para extraer incrustaciones de nodos
- **Codificador Transformer**: Mejora las representaciones de características de nodos y captura información de estructura de gráfico
- **Red Neuronal Tensorial (NTN)**: Calcula probabilidades de emparejamiento entre nodos VN y nodos SN
- **Capa de Salida**: Genera matriz de probabilidades [N_v, N_s], representando probabilidades de emparejamiento de cada nodo VN a cada nodo SN

**Estrategia de Entrenamiento:**
- **Pre-entrenamiento**: Aprende distribución de etiquetas NodeRank utilizando pérdida MSE
- **Afinación**: Optimiza la política usando el algoritmo PPO con función de recompensa basada en tasa de aceptación y utilización de recursos

## Estrategias Soportadas

- `ga`: Algoritmo Genético
- `gal-vne`: Algoritmo codicioso basado en NodeRank
- `greedy`: Algoritmo codicioso basado en ordenamiento SN
- `pretrain`: Modelo preentrenado (ft_n)
- `finetuned`: Modelo afinado (ft1)

## Configuración

La configuración principal está en `config.json`, incluyendo dimensiones del modelo, parámetros de entrenamiento, etc. Los argumentos de línea de comandos admiten configuración flexible de topología de red, tipos de flujo de trabajo, parámetros de entrenamiento, etc.

## Licencia

MIT License

---

**Nota: Este proyecto está en desarrollo activo y las APIs pueden cambiar.**
