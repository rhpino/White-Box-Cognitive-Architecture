# 📖 Guía de Implementación Paso a Paso (HOWTO)
## White-Box Cognitive Architecture (WBCA) en 4 Dimensiones

Esta guía describe el procedimiento práctico e histórico para implementar la arquitectura de Caja Blanca desde sus fases de arranque descentralizado hasta el despliegue transaccional en MariaDB con soporte temporal de 4 dimensiones (Fase C - *El Desmadre*).

---

## 🛠️ Fase A: JSON + Google Drive Multi-Dispositivo (PC vs Laptop)

### La Lógica de Negocio
Para resolver la efimeridad del contexto en las sesiones de chat de pair programming (PC y Laptop) sin depender de un servidor centralizado o base de datos en la nube costosa, se utilizó un bus de sincronización basado en carpetas compartidas de **Google Drive** persistiendo en archivos JSON planos. 

Para evitar colisiones de sobreescritura y saber "quién editó qué", cada instancia ejecutaba al arrancar una directiva `whoami` determinista, firmando cada nodo ingresado.

### Script Demostrativo: `bootstrap_whoami.py`
Guarda este script en tu directorio de desarrollo para inicializar y firmar tus nodos de conocimiento en el bus descentralizado:

```python
import os
import json
import socket
from datetime import datetime

# Definir la ruta del bus en la carpeta compartida de Google Drive
DRIVE_BUS_PATH = os.path.expanduser("~/GoogleDrive/TarsCognitiveBus/shared_knowledge.json")

def whoami():
    """Identifica dinámicamente si el agente corre en la PC de escritorio o en la Laptop."""
    hostname = socket.gethostname().lower()
    if "desktop" in hostname or "pc" in hostname:
        return "PC-Escritorio"
    elif "laptop" in hostname or "notebook" in hostname:
        return "Laptop-Movil"
    else:
        # Fallback basado en variables de entorno o genérico
        return os.getenv("DEVICE_SIGNATURE", "Equipo-Alterno")

def init_shared_bus():
    """Asegura la existencia del bus y realiza la lectura del grafo descentralizado."""
    os.makedirs(os.path.dirname(DRIVE_BUS_PATH), exist_ok=True)
    if not os.path.exists(DRIVE_BUS_PATH):
        with open(DRIVE_BUS_PATH, "w") as f:
            json.dump({"nodes": {}, "devices_seen": {}}, f, indent=4)

def register_device_presence(device_name):
    """Registra la firma del dispositivo en el bus compartido para denotar coexistencia."""
    init_shared_bus()
    
    with open(DRIVE_BUS_PATH, "r") as f:
        data = json.load(f)
        
    # Registrar presencia del equipo actual
    data["devices_seen"][device_name] = {
        "last_seen": datetime.utcnow().isoformat(),
        "status": "Online"
    }
    
    with open(DRIVE_BUS_PATH, "w") as f:
        json.dump(data, f, indent=4)
        
    print(f"✅ Dispositivo registrado en el bus: '{device_name}'")
    print("🖥️ Equipos conocidos en el bus compartido:")
    for dev, info in data["devices_seen"].items():
        print(f"  - {dev} (Último contacto: {info['last_seen']})")

def upsert_signed_node(node_id, node_data, device_name):
    """Inserta o actualiza un nodo de conocimiento firmándolo con el dispositivo autor."""
    with open(DRIVE_BUS_PATH, "r") as f:
        data = json.load(f)
        
    # Firmar y estampar temporalmente el nodo
    node_data["signature"] = device_name
    node_data["updated_at"] = datetime.utcnow().isoformat()
    
    data["nodes"][node_id] = node_data
    
    with open(DRIVE_BUS_PATH, "w") as f:
        json.dump(data, f, indent=4)
        
    print(f"📥 Nodo '{node_id}' insertado y firmado por '{device_name}'")

# Ejemplo de uso
if __name__ == "__main__":
    my_device = whoami()
    register_device_presence(my_device)
    
    upsert_signed_node(
        node_id="Router_HQ_Core",
        node_data={
            "name": "Router HQ Central",
            "ip": "192.168.1.1",
            "type": "Router"
        },
        device_name=my_device
    )
```

---

## 📄 Fase B: El Enfoque LLM-Wiki y Markdown Superdenso (M2M)

### La Lógica de Negocio
Los modelos de lenguaje no piensan como los humanos. Inyectar prosa explicativa, formateo de tablas decorativas y texto de relleno en la ventana de contexto de los LLMs consume tokens innecesarios y dispersa su foco atencional (*ruido sintáctico*).

En la Fase B, inspirados por el paradigma de *LLM-Wiki*, adoptamos la regla de **Markdown de Alta Densidad M2M (Machine-to-Machine)**. A continuación, se muestra el contraste radical de cómo se representa la misma información:

### Comparativa: Markdown Humano vs Markdown M2M

#### ❌ Markdown Tradicional (Orientado a Humanos - RUIDOSO)
```markdown
# Servidor de Monitoreo Central Zabbix
Hola. En esta página documentamos cómo está configurado el servidor de Zabbix.
El servidor físico se encuentra instalado en una máquina virtual de Proxmox en el rack del nodo HQ-Core.
Su dirección IP de administración local es la 192.168.1.5. 
Zabbix se encarga de monitorear a los routers de sucursales perimetrales y enlaces locales.

### Enlaces y Dependencias:
* Monitorea al router central del nodo HQ-Core (IP 192.168.1.1) mediante protocolo SNMP v2c.
* Monitorea la subred de la sucursal Branch-Alpha (segmento 192.168.2.0/24) a través del túnel L2TP.
* El técnico responsable de revisar las alertas es el operador asignado.
```

####  Markdown M2M Superdenso (Optimizado para LLMs - EFICIENTE)
```markdown
@node:Zabbix_Server|name="Zabbix Server"|type="Server"|ip="192.168.1.5"|host="HQ_Core_Proxmox_VM"
@edge:Zabbix_Server->Router_HQ_Core|type="monitorea_snmp"|port=161|ver="v2c"
@edge:Zabbix_Server->Subnet_Branch_Alpha|type="rutea_sobre"|segment="192.168.2.0/24"|tunnel="l2tp-branch-a"
@edge:Zabbix_Server->Operator_Zero|type="alerta_a"|role="Admin"
```

> [!NOTE]
> La versión M2M reduce el consumo de tokens atencionales en más del **75%**, eliminando conectores sintácticos innecesarios y permitiendo al modelo mapear dependencias directas mediante búsquedas regex ultra-rápidas.

---

## 🔥 Fase C (El Desmadre): MariaDB + Grafo 4D + Quórum Expandido + MCP

La fase final consolida el conocimiento en una base de datos relacional MariaDB, dotando al grafo de consistencia transaccional ACID, indexación de alta velocidad, versionamiento temporal de Elementos de Configuración (SCD Tipo 2) y un quórum inteligente de 4 agentes para auditoría total.

### 1. Inicialización de la Base de Datos Relacional: `database_setup.py`
Ejecuta este script para crear la base de datos y las tablas de grafos en 4D en tu MariaDB local o remota. Asegúrate de configurar tus credenciales de forma segura:

```python
import mysql.connector
from mysql.connector import errorcode

# NOTA: Reemplazar estas credenciales locales simuladas por tus variables de entorno seguras
db_config = {
    'user': 'db_user_example',
    'password': 'db_password_example_secure',
    'host': '127.0.0.1',
    'raise_on_warnings': True
}

def setup_database():
    try:
        conn = mysql.connector.connect(**db_config)
        cursor = conn.cursor()
        
        # 1. Crear Base de Datos
        cursor.execute("CREATE DATABASE IF NOT EXISTS wbc_cognitive_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;")
        cursor.execute("USE wbc_cognitive_db;")
        print("✅ Base de datos 'wbc_cognitive_db' inicializada.")

        # 2. Crear Tabla de Nodos Versionada
        cursor.execute("""
        CREATE TABLE IF NOT EXISTS wbc_nodes (
            node_id VARCHAR(100) NOT NULL,
            version INT NOT NULL DEFAULT 1,
            name VARCHAR(255) NOT NULL,
            type VARCHAR(50) NOT NULL,
            description TEXT,
            valid_from TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            valid_to TIMESTAMP NULL DEFAULT NULL,
            is_active TINYINT(1) NOT NULL DEFAULT 1,
            PRIMARY KEY (node_id, version),
            INDEX idx_node_active (node_id, is_active),
            INDEX idx_node_type (type)
        ) ENGINE=InnoDB;
        """)
        print("  - Tabla 'wbc_nodes' creada.")

        # 3. Crear Tabla de Aristas Versionada
        cursor.execute("""
        CREATE TABLE IF NOT EXISTS wbc_edges (
            edge_id INT AUTO_INCREMENT PRIMARY KEY,
            source_id VARCHAR(100) NOT NULL,
            source_version INT NOT NULL,
            target_id VARCHAR(100) NOT NULL,
            target_version INT NOT NULL,
            type VARCHAR(50) NOT NULL,
            attributes JSON,
            valid_from TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            valid_to TIMESTAMP NULL DEFAULT NULL,
            is_active TINYINT(1) NOT NULL DEFAULT 1,
            FOREIGN KEY (source_id, source_version) REFERENCES wbc_nodes(node_id, version) ON DELETE CASCADE,
            FOREIGN KEY (target_id, target_version) REFERENCES wbc_nodes(node_id, version) ON DELETE CASCADE,
            INDEX idx_edge_active (source_id, target_id, is_active),
            INDEX idx_edge_type (type)
        ) ENGINE=InnoDB;
        """)
        print("  - Tabla 'wbc_edges' creada.")

        # 4. Crear Tabla de Auditoría del Quórum
        cursor.execute("""
        CREATE TABLE IF NOT EXISTS cognitive_audit_log (
            audit_id INT AUTO_INCREMENT PRIMARY KEY,
            target_type ENUM('node', 'edge') NOT NULL,
            target_ref_id VARCHAR(100) NOT NULL,
            target_ref_version INT NOT NULL DEFAULT 1,
            input_payload_raw JSON,
            agent_a_opinion TEXT,
            agent_b_opinion TEXT,
            agent_c_opinion TEXT,
            agent_d_opinion TEXT,
            vote_result VARCHAR(30) NOT NULL,
            decision_rationale TEXT NOT NULL,
            model_versions JSON,
            processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            INDEX idx_audit_ref (target_ref_id, target_ref_version)
        ) ENGINE=InnoDB;
        """)
        print("  - Tabla 'cognitive_audit_log' creada.")
        
        conn.commit()
        cursor.close()
        conn.close()
        print("🎉 Configuración de base de datos completada exitosamente.")
        
    except mysql.connector.Error as err:
        print(f"❌ Error al configurar base de datos: {err}")

if __name__ == "__main__":
    setup_database()
```

---

### 2. El Daemon de Ingesta y Optimización FinOps: `ingest_daemon_4d.py`
Este script demuestra cómo opera el flujo del daemon asíncrono. Implementa el **Fast-Path** por umbral de confianza y el enrutamiento de **Deep-Debate con 4 Agentes** en caso de colisión histórica en la base de datos.

```python
import json
import random
import time

# Datos simulados de la red de borde entrantes (Payload Crudo Genérico)
SAMPLE_PAYLOAD_EASY = {
    "node_id": "Router_Branch_Alpha",
    "name": "Router de Borde Branch-Alpha",
    "type": "Router",
    "ip": "192.168.2.1",
    "edges": [
        {"target_id": "Router_HQ_Core", "type": "tunel_l2tp", "attributes": {"port": 1701}}
    ]
}

SAMPLE_PAYLOAD_ANOMALY = {
    "node_id": "Router_HQ_Core", # ¡Ojo! Intentando asociar una IP crítica del servidor de monitoreo
    "name": "Router Central HQ-Core",
    "type": "Router",
    "ip": "192.168.1.5", # ¡Colisión! IP pertenece originalmente a Zabbix Server
    "edges": [
        {"target_id": "Zabbix_Server", "type": "conecta_con", "attributes": {"port": 10051}}
    ]
}

def evaluate_structural_confidence(payload):
    """
    Simula el Clasificador Flash inicial.
    Retorna score de confianza (0.0 a 1.0) y la propuesta estructurada.
    """
    if payload.get("node_id") and payload.get("type") and payload.get("ip"):
        return 0.98, payload
    return 0.70, payload

def check_db_history_collision(payload):
    """
    Comprueba si el payload intenta reescribir una IP crítica que pertenecía
    anteriormente a otro dispositivo en el historial del grafo.
    """
    # Simulación de colisión histórica local de rango privado
    if payload.get("ip") == "192.168.1.5" and payload.get("node_id") != "Zabbix_Server":
        return True # ¡Colisión histórica detectada!
    return False

def run_quorum_4d(payload):
    """Simula el debate profundo con los Agentes A, B, C y D."""
    print("⏳ Activando DEEP-DEBATE PATH (Quórum de 4 Agentes)...")
    time.sleep(1)
    
    opinion_a = f"Agente A (Analista): Entidad identificada '{payload['node_id']}' de tipo '{payload['type']}' con IP '{payload['ip']}'."
    opinion_b = f"Agente B (Compilador): Diseñadas {len(payload['edges'])} aristas estáticas hacia {payload['edges']}."
    opinion_c = "Agente C (CISO): VETO - Violación detectada. La IP 192.168.1.5 está reservada históricamente para Zabbix Server!"
    opinion_d = "Agente D (El Historiador): Anomalía confirmada. Hay un conflicto de usurpación de IP con la versión 1 del Zabbix Server en MariaDB."
    
    return {
        "A": opinion_a,
        "B": opinion_b,
        "C": opinion_c,
        "D": opinion_d,
        "vote_result": "4-0-REJECTED",
        "approved": False,
        "rationale": "Intento de usurpación de dirección IP de Zabbix Server en el núcleo de red."
    }

def process_pipeline(payload):
    print(f"\n📥 Recibido nuevo payload de infraestructura para nodo: '{payload['node_id']}'")
    
    # 1. Evaluación de Confianza Estructural
    score, proposal = evaluate_structural_confidence(payload)
    
    # 2. Control de Anomalías Históricas
    collision = check_db_history_collision(payload)
    
    # 3. Decision de Enrutamiento (FinOps & Security Bypass)
    if score >= 0.95 and not collision:
        # FAST-PATH: Compilación directa para ahorrar tokens
        print("⚡ FAST-PATH activado (Confianza estructural alta sin colisiones históricas).")
        vote_result = "FAST-PATH"
        approved = True
        rationale = "Compilado de forma automática."
        print(f"💾 Insertando versión en wbc_nodes y wbc_edges transaccionalmente.")
    else:
        # DEEP-DEBATE PATH: Invocación del Quórum Expandido 4D
        debate = run_quorum_4d(payload)
        vote_result = debate["vote_result"]
        approved = debate["approved"]
        rationale = debate["rationale"]
        
    # 4. Grabado en log de auditoría
    print(f"📝 Grabado en 'cognitive_audit_log' -> Resultado: {vote_result} | Aprobado: {approved}")
    print(f"💡 Rationale de Decisión: {rationale}")

if __name__ == "__main__":
    # Test 1: Flujo Rápido (Fast-Path)
    process_pipeline(SAMPLE_PAYLOAD_EASY)
    
    # Test 2: Bloqueo de Anomalía Temporal (Deep-Debate)
    process_pipeline(SAMPLE_PAYLOAD_ANOMALY)
```

---

### 3. Servidor de Consultas MCP Temporal: `mcp_tools_demo.py`
Este script demuestra cómo un agente de producción o un dashboard interroga al servidor MCP para de-compilar el grafo relacional 4D de MariaDB y realizar **viajes en el tiempo** para diagnosticar problemas basándose en deltas temporales.

```python
import json
from datetime import datetime

def query_decompiler_delta(node_id, target_time):
    """
    Simula la consulta SQL que realiza el servidor MCP a MariaDB 
    buscando qué versión del Elemento de Configuración estaba activa 
    en un instante exacto de tiempo.
    """
    
    # Simulamos el estado histórico de la base de datos local
    db_history = {
        "Router_Branch_Alpha": [
            {
                "version": 1,
                "name": "Router de Borde Branch-Alpha",
                "ip": "192.168.2.1",
                "vpn_tunnel": "WireGuard",
                "valid_from": "2026-05-22T08:00:00Z",
                "valid_to": "2026-05-22T21:27:00Z",
                "is_active": 0
            },
            {
                "version": 2,
                "name": "Router de Borde Branch-Alpha",
                "ip": "192.168.2.1",
                "vpn_tunnel": "SSTP", # Cambio de túnel en caliente
                "valid_from": "2026-05-22T21:27:00Z",
                "valid_to": None,
                "is_active": 1
            }
        ]
    }
    
    records = db_history.get(node_id, [])
    active_version = None
    
    # Parsear hora de consulta
    query_dt = datetime.strptime(target_time, "%Y-%m-%dT%H:%M:%SZ")
    
    for r in records:
        from_dt = datetime.strptime(r["valid_from"], "%Y-%m-%dT%H:%M:%SZ")
        to_dt = datetime.strptime(r["valid_to"], "%Y-%m-%dT%H:%M:%SZ") if r["valid_to"] else datetime.max
        
        if from_dt <= query_dt <= to_dt:
            active_version = r
            break
            
    # El Agente Decompiler traduce el delta del grafo a prosa humana
    if active_version:
        prosa_decompiler = f"El dispositivo '{active_version['name']}' (Versión {active_version['version']}) " \
                           f"operaba con el túnel VPN '{active_version['vpn_tunnel']}' e IP '{active_version['ip']}' " \
                           f"en la fecha consultada ({target_time})."
        return prosa_decompiler
    return "Ninguna configuración activa encontrada para esa fecha."

if __name__ == "__main__":
    print("🔎 Interrogando al Servidor MCP en dos puntos de la línea de tiempo:")
    
    # Consulta 1: Antes del mantenimiento
    print("\n⏱️ Hora: 2026-05-22T10:00:00Z")
    print(query_decompiler_delta("Router_Branch_Alpha", "2026-05-22T10:00:00Z"))
    
    # Consulta 2: Después del mantenimiento
    print("\n⏱️ Hora: 2026-05-22T22:00:00Z")
    print(query_decompiler_delta("Router_Branch_Alpha", "2026-05-22T22:00:00Z"))
```

---

Con estos scripts base, tienes los planos y los kernels necesarios para levantar tu propio **Grafo Temporal Cognitivo de 4 Dimensiones** en MariaDB, protegiendo para siempre la ventana de contexto de tus agentes de IA.
