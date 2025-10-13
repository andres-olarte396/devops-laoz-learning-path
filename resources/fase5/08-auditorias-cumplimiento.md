# Módulo 08: Auditorías y Cumplimiento

## 📋 Introducción

Las auditorías y el cumplimiento normativo son aspectos críticos en entornos DevOps modernos. Las organizaciones deben cumplir con regulaciones como SOX, PCI-DSS, GDPR, HIPAA, y estándares como ISO 27001. Este módulo cubre cómo implementar controles de auditoría automatizados y mantener el cumplimiento continuo.

### ¿Qué es Compliance en DevOps?

Compliance en DevOps significa:

- **Automatizar** controles de cumplimiento en pipelines
- **Documentar** todos los cambios y accesos
- **Monitorear** continuamente la adherencia a políticas
- **Reportar** el estado de cumplimiento en tiempo real
- **Remediar** automáticamente desviaciones

### Marcos de Cumplimiento Comunes

- **SOX (Sarbanes-Oxley)**: Controles financieros y segregación de funciones
- **PCI-DSS**: Seguridad de datos de tarjetas de pago
- **GDPR**: Protección de datos personales en EU
- **HIPAA**: Protección de información de salud
- **ISO 27001**: Sistema de gestión de seguridad de la información
- **SOC 2**: Controles de seguridad, disponibilidad e integridad

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Implementar** controles de auditoría automatizados
2. **Configurar** logging y monitoreo para compliance
3. **Crear** reportes de cumplimiento automatizados
4. **Implementar** segregación de funciones (SOD)
5. **Configurar** controles de acceso y privilegios
6. **Automatizar** validación de configuraciones
7. **Gestionar** evidencias de auditoría
8. **Implementar** compliance as code

---

## 🏗️ Arquitectura de Compliance

### 1. Stack de Auditoría

```yaml
# docker-compose.compliance.yml
version: '3.8'

services:
  # Elasticsearch para logs de auditoría
  elasticsearch:
    image: elasticsearch:8.11.0
    container_name: compliance-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=true
      - ELASTIC_PASSWORD=elastic123
      - xpack.security.audit.enabled=true
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
      - ./elasticsearch/config:/usr/share/elasticsearch/config
    ports:
      - "9200:9200"
    networks:
      - compliance_network

  # Logstash para procesamiento de logs
  logstash:
    image: logstash:8.11.0
    container_name: compliance-logstash
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5044:5044"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx512m -Xms512m"
    networks:
      - compliance_network
    depends_on:
      - elasticsearch

  # Kibana para visualización
  kibana:
    image: kibana:8.11.0
    container_name: compliance-kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
      ELASTICSEARCH_USERNAME: elastic
      ELASTICSEARCH_PASSWORD: elastic123
    volumes:
      - ./kibana/config:/usr/share/kibana/config
    networks:
      - compliance_network
    depends_on:
      - elasticsearch

  # Falco para runtime security monitoring
  falco:
    image: falcosecurity/falco:0.36.2
    container_name: compliance-falco
    privileged: true
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
      - /etc:/host/etc:ro
      - ./falco/rules:/etc/falco/rules.d
    command:
      - /usr/bin/falco
      - --cri
      - /run/containerd/containerd.sock
      - -K
      - /var/run/secrets/kubernetes.io/serviceaccount/token
      - -k
      - https://kubernetes.default
      - -pk
    networks:
      - compliance_network

  # Open Policy Agent para policy enforcement
  opa:
    image: openpolicyagent/opa:latest-envoy
    container_name: compliance-opa
    ports:
      - "8181:8181"
    volumes:
      - ./opa/policies:/policies
    command:
      - "run"
      - "--server"
      - "--addr=0.0.0.0:8181"
      - "--log-level=debug"
      - "/policies"
    networks:
      - compliance_network

  # Grafana para dashboards de compliance
  grafana:
    image: grafana/grafana:10.2.0
    container_name: compliance-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    networks:
      - compliance_network

  # Vault para audit logging
  vault:
    image: hashicorp/vault:1.15.2
    container_name: compliance-vault
    ports:
      - "8200:8200"
    environment:
      VAULT_LOCAL_CONFIG: |
        {
          "backend": {"file": {"path": "/vault/file"}},
          "listener": {"tcp": {"address": "0.0.0.0:8200", "tls_disable": true}},
          "default_lease_ttl": "168h",
          "max_lease_ttl": "720h",
          "ui": true,
          "log_level": "Info",
          "audit": {
            "file": {
              "type": "file",
              "options": {
                "file_path": "/vault/logs/audit.log"
              }
            }
          }
        }
    cap_add:
      - IPC_LOCK
    volumes:
      - vault_data:/vault/file
      - vault_logs:/vault/logs
    networks:
      - compliance_network

volumes:
  elasticsearch_data:
  grafana_data:
  vault_data:
  vault_logs:

networks:
  compliance_network:
    driver: bridge
```

### 2. Configuración de Logstash para Auditoría

```ruby
# logstash/pipeline/audit.conf
input {
  # Logs de aplicación
  beats {
    port => 5044
  }
  
  # Logs de Kubernetes audit
  file {
    path => "/var/log/kubernetes/audit.log"
    start_position => "beginning"
    codec => "json"
    tags => ["kubernetes", "audit"]
  }
  
  # Logs de Vault audit
  file {
    path => "/vault/logs/audit.log"
    start_position => "beginning"
    codec => "json"
    tags => ["vault", "audit"]
  }
  
  # Logs de sistema
  syslog {
    port => 514
    tags => ["syslog", "system"]
  }
}

filter {
  # Procesar logs de Kubernetes audit
  if "kubernetes" in [tags] and "audit" in [tags] {
    mutate {
      add_field => { "compliance_category" => "kubernetes_audit" }
    }
    
    # Clasificar por nivel de riesgo
    if [verb] in ["create", "update", "patch", "delete"] {
      mutate {
        add_field => { "risk_level" => "high" }
      }
    } else if [verb] in ["get", "list", "watch"] {
      mutate {
        add_field => { "risk_level" => "low" }
      }
    } else {
      mutate {
        add_field => { "risk_level" => "medium" }
      }
    }
    
    # Detectar operaciones sensibles
    if [objectRef][resource] in ["secrets", "configmaps"] or
       [objectRef][namespace] in ["kube-system", "kube-public"] {
      mutate {
        add_field => { "sensitive_operation" => "true" }
      }
    }
  }
  
  # Procesar logs de Vault audit
  if "vault" in [tags] and "audit" in [tags] {
    mutate {
      add_field => { "compliance_category" => "vault_audit" }
    }
    
    # Clasificar operaciones de Vault
    if [request][operation] in ["create", "update", "delete"] {
      mutate {
        add_field => { "risk_level" => "high" }
      }
    } else if [request][operation] == "read" {
      mutate {
        add_field => { "risk_level" => "medium" }
      }
    } else {
      mutate {
        add_field => { "risk_level" => "low" }
      }
    }
    
    # Detectar acceso a secretos críticos
    if [request][path] =~ /secret\/prod\// or [request][path] =~ /database\/creds/ {
      mutate {
        add_field => { "sensitive_operation" => "true" }
      }
    }
  }
  
  # Enriquecer con información geográfica
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
  
  # Detectar patrones anómalos
  if [user] {
    # Detectar usuarios con múltiples IPs
    aggregate {
      task_id => "%{user}"
      code => "
        map['user'] = event.get('user')
        map['ips'] ||= []
        map['ips'] << event.get('client_ip') if event.get('client_ip')
        map['ips'].uniq!
        event.set('unique_ips_count', map['ips'].length)
        if map['ips'].length > 3
          event.set('anomaly_multiple_ips', true)
        end
      "
      push_map_as_event_on_timeout => false
      timeout => 300
      inactivity_timeout => 60
    }
  }
  
  # Añadir timestamp de procesamiento
  ruby {
    code => "event.set('processed_at', Time.now.utc.iso8601)"
  }
}

output {
  # Enviar a Elasticsearch
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    user => "elastic"
    password => "elastic123"
    index => "compliance-audit-%{+YYYY.MM.dd}"
    template_name => "compliance-audit"
    template => "/usr/share/logstash/templates/compliance-audit.json"
    template_overwrite => true
  }
  
  # Alertas críticas a Slack
  if [risk_level] == "high" and [sensitive_operation] == "true" {
    http {
      url => "${SLACK_WEBHOOK_URL}"
      http_method => "post"
      content_type => "application/json"
      format => "json"
      mapping => {
        "text" => "🚨 Critical compliance event detected"
        "attachments" => [
          {
            "color" => "danger"
            "title" => "Compliance Alert"
            "fields" => [
              {
                "title" => "User"
                "value" => "%{user}"
                "short" => true
              },
              {
                "title" => "Operation"
                "value" => "%{[request][operation]}"
                "short" => true
              },
              {
                "title" => "Resource"
                "value" => "%{[request][path]}"
                "short" => false
              },
              {
                "title" => "Timestamp"
                "value" => "%{@timestamp}"
                "short" => true
              }
            ]
          }
        ]
      }
    }
  }
  
  # Debug output
  if [compliance_category] {
    stdout {
      codec => rubydebug {
        metadata => false
      }
    }
  }
}
```

---

## 🔍 Implementación de Controles SOX

### 1. Segregación de Funciones (SOD)

```python
#!/usr/bin/env python3
# compliance/sox_controls.py

import json
import logging
from typing import Dict, List, Set, Tuple
from dataclasses import dataclass
from datetime import datetime, timedelta
import requests

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dataclass
class User:
    username: str
    roles: Set[str]
    permissions: Set[str]
    department: str
    manager: str

@dataclass
class SODViolation:
    user: str
    conflicting_roles: List[str]
    violation_type: str
    risk_level: str
    description: str
    detected_at: datetime

class SOXComplianceChecker:
    def __init__(self):
        self.sod_rules = self.load_sod_rules()
        self.violations = []
        
    def load_sod_rules(self) -> Dict:
        """Cargar reglas de segregación de funciones"""
        return {
            "financial_controls": {
                "incompatible_roles": [
                    ["financial_approver", "payment_processor"],
                    ["invoice_creator", "invoice_approver"],
                    ["journal_entry_creator", "journal_entry_approver"],
                    ["budget_creator", "budget_approver"]
                ],
                "description": "Financial processes require separation of duties"
            },
            "it_controls": {
                "incompatible_roles": [
                    ["developer", "production_deployer"],
                    ["security_admin", "system_admin"],
                    ["database_admin", "application_admin"],
                    ["change_requestor", "change_approver"]
                ],
                "description": "IT processes require separation of duties"
            },
            "access_controls": {
                "incompatible_roles": [
                    ["user_provisioner", "access_reviewer"],
                    ["role_creator", "role_assigner"],
                    ["audit_reviewer", "system_admin"]
                ],
                "description": "Access management requires separation of duties"
            }
        }
    
    def check_user_sod_violations(self, user: User) -> List[SODViolation]:
        """Verificar violaciones SOD para un usuario"""
        violations = []
        
        for control_category, rules in self.sod_rules.items():
            for incompatible_pair in rules["incompatible_roles"]:
                if all(role in user.roles for role in incompatible_pair):
                    violation = SODViolation(
                        user=user.username,
                        conflicting_roles=incompatible_pair,
                        violation_type="segregation_of_duties",
                        risk_level="high",
                        description=f"User has incompatible roles: {', '.join(incompatible_pair)}",
                        detected_at=datetime.now()
                    )
                    violations.append(violation)
                    logger.warning(f"SOD violation detected for {user.username}: {incompatible_pair}")
        
        return violations
    
    def check_all_users_sod(self, users: List[User]) -> List[SODViolation]:
        """Verificar violaciones SOD para todos los usuarios"""
        all_violations = []
        
        for user in users:
            user_violations = self.check_user_sod_violations(user)
            all_violations.extend(user_violations)
        
        return all_violations
    
    def generate_sod_report(self, violations: List[SODViolation]) -> Dict:
        """Generar reporte de violaciones SOD"""
        report = {
            "report_date": datetime.now().isoformat(),
            "total_violations": len(violations),
            "violations_by_type": {},
            "violations_by_risk": {},
            "violations": []
        }
        
        for violation in violations:
            # Contar por tipo
            if violation.violation_type not in report["violations_by_type"]:
                report["violations_by_type"][violation.violation_type] = 0
            report["violations_by_type"][violation.violation_type] += 1
            
            # Contar por riesgo
            if violation.risk_level not in report["violations_by_risk"]:
                report["violations_by_risk"][violation.risk_level] = 0
            report["violations_by_risk"][violation.risk_level] += 1
            
            # Añadir detalle
            report["violations"].append({
                "user": violation.user,
                "conflicting_roles": violation.conflicting_roles,
                "violation_type": violation.violation_type,
                "risk_level": violation.risk_level,
                "description": violation.description,
                "detected_at": violation.detected_at.isoformat()
            })
        
        return report

class ITGCControls:
    """IT General Controls para SOX compliance"""
    
    def __init__(self):
        self.controls = self.define_itgc_controls()
    
    def define_itgc_controls(self) -> Dict:
        """Definir controles ITGC"""
        return {
            "access_controls": {
                "AC01": {
                    "description": "User access provisioning requires approval",
                    "control_type": "preventive",
                    "frequency": "per_transaction",
                    "owner": "security_team"
                },
                "AC02": {
                    "description": "Periodic access reviews are performed",
                    "control_type": "detective",
                    "frequency": "quarterly",
                    "owner": "security_team"
                },
                "AC03": {
                    "description": "Privileged access is monitored",
                    "control_type": "detective",
                    "frequency": "continuous",
                    "owner": "security_team"
                }
            },
            "change_management": {
                "CM01": {
                    "description": "All changes require approval",
                    "control_type": "preventive",
                    "frequency": "per_transaction",
                    "owner": "change_advisory_board"
                },
                "CM02": {
                    "description": "Emergency changes are reviewed post-implementation",
                    "control_type": "detective",
                    "frequency": "per_occurrence",
                    "owner": "change_advisory_board"
                },
                "CM03": {
                    "description": "Production changes are logged and monitored",
                    "control_type": "detective",
                    "frequency": "continuous",
                    "owner": "operations_team"
                }
            },
            "data_backup": {
                "DB01": {
                    "description": "Critical data is backed up daily",
                    "control_type": "preventive",
                    "frequency": "daily",
                    "owner": "operations_team"
                },
                "DB02": {
                    "description": "Backup integrity is tested monthly",
                    "control_type": "detective",
                    "frequency": "monthly",
                    "owner": "operations_team"
                }
            }
        }
    
    def test_control_effectiveness(self, control_id: str, evidence: Dict) -> Dict:
        """Probar efectividad de un control"""
        if control_id not in self.get_all_control_ids():
            return {"status": "error", "message": "Control ID not found"}
        
        control = self.get_control_by_id(control_id)
        test_result = {
            "control_id": control_id,
            "test_date": datetime.now().isoformat(),
            "control_description": control["description"],
            "evidence_provided": bool(evidence),
            "test_status": "pass",  # Default
            "findings": [],
            "recommendations": []
        }
        
        # Lógica específica de testing por tipo de control
        if control_id.startswith("AC"):  # Access Controls
            test_result = self._test_access_control(control_id, evidence, test_result)
        elif control_id.startswith("CM"):  # Change Management
            test_result = self._test_change_management(control_id, evidence, test_result)
        elif control_id.startswith("DB"):  # Data Backup
            test_result = self._test_data_backup(control_id, evidence, test_result)
        
        return test_result
    
    def _test_access_control(self, control_id: str, evidence: Dict, test_result: Dict) -> Dict:
        """Probar controles de acceso"""
        if control_id == "AC01":  # User provisioning approval
            if "approval_tickets" not in evidence:
                test_result["test_status"] = "fail"
                test_result["findings"].append("No approval tickets provided for user provisioning")
            elif len(evidence["approval_tickets"]) == 0:
                test_result["test_status"] = "fail"
                test_result["findings"].append("No approved provisioning requests found")
        
        elif control_id == "AC02":  # Periodic access reviews
            if "access_review_reports" not in evidence:
                test_result["test_status"] = "fail"
                test_result["findings"].append("No access review reports provided")
            else:
                # Verificar que las revisiones sean trimestrales
                last_review = evidence.get("last_review_date")
                if last_review:
                    from dateutil.parser import parse
                    last_review_date = parse(last_review)
                    if (datetime.now() - last_review_date).days > 90:
                        test_result["test_status"] = "fail"
                        test_result["findings"].append("Access review overdue (>90 days)")
        
        return test_result
    
    def _test_change_management(self, control_id: str, evidence: Dict, test_result: Dict) -> Dict:
        """Probar controles de gestión de cambios"""
        if control_id == "CM01":  # Change approval
            if "change_tickets" not in evidence:
                test_result["test_status"] = "fail"
                test_result["findings"].append("No change tickets provided")
            else:
                unapproved_changes = [
                    ticket for ticket in evidence["change_tickets"]
                    if ticket.get("status") != "approved"
                ]
                if unapproved_changes:
                    test_result["test_status"] = "fail"
                    test_result["findings"].append(f"{len(unapproved_changes)} unapproved changes detected")
        
        return test_result
    
    def _test_data_backup(self, control_id: str, evidence: Dict, test_result: Dict) -> Dict:
        """Probar controles de backup de datos"""
        if control_id == "DB01":  # Daily backups
            if "backup_logs" not in evidence:
                test_result["test_status"] = "fail"
                test_result["findings"].append("No backup logs provided")
            else:
                # Verificar backups diarios
                backup_dates = evidence["backup_logs"]
                today = datetime.now().date()
                if str(today) not in backup_dates:
                    test_result["test_status"] = "fail"
                    test_result["findings"].append("No backup performed today")
        
        return test_result
    
    def get_all_control_ids(self) -> List[str]:
        """Obtener todos los IDs de controles"""
        control_ids = []
        for category in self.controls.values():
            control_ids.extend(category.keys())
        return control_ids
    
    def get_control_by_id(self, control_id: str) -> Dict:
        """Obtener control por ID"""
        for category in self.controls.values():
            if control_id in category:
                return category[control_id]
        return {}

# Ejemplo de uso
def main():
    # Datos de prueba
    users = [
        User("john.doe", {"developer", "production_deployer"}, set(), "IT", "jane.manager"),
        User("jane.smith", {"financial_approver", "payment_processor"}, set(), "Finance", "bob.director"),
        User("bob.wilson", {"audit_reviewer"}, set(), "Audit", "alice.cfo")
    ]
    
    # Verificar violaciones SOD
    sox_checker = SOXComplianceChecker()
    violations = sox_checker.check_all_users_sod(users)
    report = sox_checker.generate_sod_report(violations)
    
    print("SOX Compliance Report:")
    print(json.dumps(report, indent=2))
    
    # Probar controles ITGC
    itgc = ITGCControls()
    
    # Evidencia de prueba para AC01
    evidence = {
        "approval_tickets": [
            {"ticket_id": "REQ-001", "status": "approved", "approver": "security_manager"},
            {"ticket_id": "REQ-002", "status": "approved", "approver": "security_manager"}
        ]
    }
    
    test_result = itgc.test_control_effectiveness("AC01", evidence)
    print("\nITGC Control Test Result:")
    print(json.dumps(test_result, indent=2))

if __name__ == "__main__":
    main()
```

---

## 📊 Controles PCI-DSS

### 1. Monitoreo de Seguridad de Tarjetas

```yaml
# compliance/pci-dss-monitoring.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pci-dss-falco-rules
data:
  pci_rules.yaml: |
    - rule: PCI - Unauthorized access to cardholder data
      desc: Detect unauthorized access to directories containing cardholder data
      condition: >
        open_read and (
          fd.name startswith /var/cardholder_data/ or
          fd.name startswith /opt/payment_data/
        ) and not proc.name in (authorized_processes)
      output: >
        Unauthorized access to cardholder data 
        (user=%user.name command=%proc.cmdline file=%fd.name)
      priority: CRITICAL
      tags: [pci-dss, cardholder_data, unauthorized_access]

    - rule: PCI - Credit card pattern in logs
      desc: Detect potential credit card numbers in log files
      condition: >
        spawned_process and (
          proc.cmdline contains "4[0-9]{12}(?:[0-9]{3})?" or
          proc.cmdline contains "5[1-5][0-9]{14}" or
          proc.cmdline contains "3[47][0-9]{13}"
        )
      output: >
        Potential credit card number detected in process 
        (user=%user.name command=%proc.cmdline)
      priority: HIGH
      tags: [pci-dss, credit_card, data_exposure]

    - rule: PCI - Network access to payment systems
      desc: Monitor network connections to payment processing systems
      condition: >
        outbound and (
          fd.rip in (payment_server_ips) or
          fd.rport in (payment_ports)
        ) and not proc.name in (authorized_payment_processes)
      output: >
        Unauthorized network access to payment systems 
        (user=%user.name process=%proc.name connection=%fd.name)
      priority: HIGH
      tags: [pci-dss, network_access, payment_systems]

    - rule: PCI - Database access outside business hours
      desc: Detect database access outside of business hours
      condition: >
        spawned_process and proc.name in (mysql, psql, sqlplus) and
        not (hour >= 8 and hour <= 18)
      output: >
        Database access outside business hours 
        (user=%user.name time=%proc.atime command=%proc.cmdline)
      priority: MEDIUM
      tags: [pci-dss, database_access, after_hours]

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pci-dss-monitor
  labels:
    app: pci-dss-monitor
spec:
  selector:
    matchLabels:
      app: pci-dss-monitor
  template:
    metadata:
      labels:
        app: pci-dss-monitor
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: falco
        image: falcosecurity/falco:0.36.2
        args:
          - /usr/bin/falco
          - --cri
          - /var/run/containerd/containerd.sock
          - -r
          - /etc/falco/rules.d/pci_rules.yaml
        volumeMounts:
        - name: pci-rules
          mountPath: /etc/falco/rules.d
        - name: var-run
          mountPath: /var/run
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: boot
          mountPath: /host/boot
          readOnly: true
        - name: modules
          mountPath: /host/lib/modules
          readOnly: true
        - name: usr
          mountPath: /host/usr
          readOnly: true
        securityContext:
          privileged: true
      volumes:
      - name: pci-rules
        configMap:
          name: pci-dss-falco-rules
      - name: var-run
        hostPath:
          path: /var/run
      - name: proc
        hostPath:
          path: /proc
      - name: boot
        hostPath:
          path: /boot
      - name: modules
        hostPath:
          path: /lib/modules
      - name: usr
        hostPath:
          path: /usr
```

### 2. Data Loss Prevention (DLP)

```python
#!/usr/bin/env python3
# compliance/dlp_scanner.py

import re
import os
import json
import hashlib
from typing import Dict, List, Tuple
from dataclasses import dataclass
from datetime import datetime
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dataclass
class DLPViolation:
    file_path: str
    line_number: int
    violation_type: str
    pattern_matched: str
    confidence: str
    severity: str
    timestamp: datetime

class DLPScanner:
    def __init__(self):
        self.patterns = self.load_patterns()
        self.violations = []
    
    def load_patterns(self) -> Dict:
        """Cargar patrones de detección de datos sensibles"""
        return {
            "credit_cards": {
                "patterns": [
                    r'\b4[0-9]{12}(?:[0-9]{3})?\b',  # Visa
                    r'\b5[1-5][0-9]{14}\b',          # MasterCard
                    r'\b3[47][0-9]{13}\b',           # American Express
                    r'\b6(?:011|5[0-9]{2})[0-9]{12}\b'  # Discover
                ],
                "description": "Credit card numbers",
                "severity": "critical"
            },
            "ssn": {
                "patterns": [
                    r'\b\d{3}-?\d{2}-?\d{4}\b',
                    r'\b\d{9}\b'
                ],
                "description": "Social Security Numbers",
                "severity": "high"
            },
            "email_addresses": {
                "patterns": [
                    r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
                ],
                "description": "Email addresses",
                "severity": "medium"
            },
            "phone_numbers": {
                "patterns": [
                    r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b',
                    r'\(\d{3}\)\s*\d{3}[-.]?\d{4}\b'
                ],
                "description": "Phone numbers",
                "severity": "low"
            },
            "api_keys": {
                "patterns": [
                    r'[Aa][Pp][Ii]_?[Kk][Ee][Yy]\s*[:=]\s*["\']?([A-Za-z0-9_-]{20,})["\']?',
                    r'[Ss][Ee][Cc][Rr][Ee][Tt]_?[Kk][Ee][Yy]\s*[:=]\s*["\']?([A-Za-z0-9_-]{20,})["\']?',
                    r'[Aa][Cc][Cc][Ee][Ss][Ss]_?[Tt][Oo][Kk][Ee][Nn]\s*[:=]\s*["\']?([A-Za-z0-9_-]{20,})["\']?'
                ],
                "description": "API keys and tokens",
                "severity": "high"
            },
            "database_connections": {
                "patterns": [
                    r'[Pp][Aa][Ss][Ss][Ww][Oo][Rr][Dd]\s*[:=]\s*["\']?([^"\'\s]+)["\']?',
                    r'[Dd][Bb]_?[Pp][Aa][Ss][Ss]\s*[:=]\s*["\']?([^"\'\s]+)["\']?',
                    r'mysql://[^:\s]+:([^@\s]+)@',
                    r'postgresql://[^:\s]+:([^@\s]+)@'
                ],
                "description": "Database credentials",
                "severity": "critical"
            }
        }
    
    def scan_file(self, file_path: str) -> List[DLPViolation]:
        """Escanear un archivo en busca de datos sensibles"""
        violations = []
        
        try:
            with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                for line_num, line in enumerate(f, 1):
                    line_violations = self.scan_line(file_path, line_num, line)
                    violations.extend(line_violations)
        except Exception as e:
            logger.error(f"Error scanning file {file_path}: {e}")
        
        return violations
    
    def scan_line(self, file_path: str, line_num: int, line: str) -> List[DLPViolation]:
        """Escanear una línea en busca de patrones sensibles"""
        violations = []
        
        for pattern_type, config in self.patterns.items():
            for pattern in config["patterns"]:
                matches = re.finditer(pattern, line, re.IGNORECASE)
                for match in matches:
                    # Validar si es un verdadero positivo
                    confidence = self.calculate_confidence(pattern_type, match.group())
                    
                    if confidence != "false_positive":
                        violation = DLPViolation(
                            file_path=file_path,
                            line_number=line_num,
                            violation_type=pattern_type,
                            pattern_matched=match.group(),
                            confidence=confidence,
                            severity=config["severity"],
                            timestamp=datetime.now()
                        )
                        violations.append(violation)
                        
                        logger.warning(f"DLP violation: {pattern_type} in {file_path}:{line_num}")
        
        return violations
    
    def calculate_confidence(self, pattern_type: str, matched_text: str) -> str:
        """Calcular confianza del match para reducir falsos positivos"""
        
        if pattern_type == "credit_cards":
            # Validar usando algoritmo de Luhn
            return self.validate_credit_card(matched_text)
        
        elif pattern_type == "ssn":
            # Validar formato de SSN
            clean_ssn = re.sub(r'[^\d]', '', matched_text)
            if len(clean_ssn) == 9:
                # Verificar que no sea un patrón obvio (111-11-1111, etc.)
                if clean_ssn == clean_ssn[0] * 9:
                    return "false_positive"
                return "high"
            return "medium"
        
        elif pattern_type == "email_addresses":
            # Verificar que no sea un ejemplo genérico
            generic_patterns = ['example.com', 'test.com', 'localhost', '127.0.0.1']
            if any(generic in matched_text.lower() for generic in generic_patterns):
                return "false_positive"
            return "high"
        
        elif pattern_type == "api_keys":
            # Verificar longitud y complejidad
            if len(matched_text) < 16:
                return "low"
            elif len(matched_text) > 40:
                return "high"
            return "medium"
        
        return "medium"
    
    def validate_credit_card(self, card_number: str) -> str:
        """Validar número de tarjeta usando algoritmo de Luhn"""
        # Limpiar número
        card_number = re.sub(r'[^\d]', '', card_number)
        
        if len(card_number) < 13 or len(card_number) > 19:
            return "false_positive"
        
        # Algoritmo de Luhn
        def luhn_check(card_num):
            def digits_of(n):
                return [int(d) for d in str(n)]
            
            digits = digits_of(card_num)
            odd_digits = digits[-1::-2]
            even_digits = digits[-2::-2]
            checksum = sum(odd_digits)
            
            for d in even_digits:
                checksum += sum(digits_of(d * 2))
            
            return checksum % 10 == 0
        
        if luhn_check(card_number):
            return "high"
        else:
            return "false_positive"
    
    def scan_directory(self, directory: str, extensions: List[str] = None) -> List[DLPViolation]:
        """Escanear directorio recursivamente"""
        if extensions is None:
            extensions = ['.py', '.js', '.java', '.cs', '.php', '.rb', '.go', '.sql', '.txt', '.log', '.config', '.json', '.xml', '.yaml', '.yml']
        
        all_violations = []
        
        for root, dirs, files in os.walk(directory):
            # Excluir directorios comunes que no deben escanearse
            dirs[:] = [d for d in dirs if d not in ['.git', 'node_modules', '__pycache__', '.venv']]
            
            for file in files:
                if any(file.endswith(ext) for ext in extensions):
                    file_path = os.path.join(root, file)
                    file_violations = self.scan_file(file_path)
                    all_violations.extend(file_violations)
        
        return all_violations
    
    def generate_report(self, violations: List[DLPViolation]) -> Dict:
        """Generar reporte de violaciones DLP"""
        report = {
            "scan_date": datetime.now().isoformat(),
            "total_violations": len(violations),
            "violations_by_type": {},
            "violations_by_severity": {},
            "violations_by_confidence": {},
            "high_risk_files": [],
            "violations": []
        }
        
        file_violation_count = {}
        
        for violation in violations:
            # Contar por tipo
            if violation.violation_type not in report["violations_by_type"]:
                report["violations_by_type"][violation.violation_type] = 0
            report["violations_by_type"][violation.violation_type] += 1
            
            # Contar por severidad
            if violation.severity not in report["violations_by_severity"]:
                report["violations_by_severity"][violation.severity] = 0
            report["violations_by_severity"][violation.severity] += 1
            
            # Contar por confianza
            if violation.confidence not in report["violations_by_confidence"]:
                report["violations_by_confidence"][violation.confidence] = 0
            report["violations_by_confidence"][violation.confidence] += 1
            
            # Contar violaciones por archivo
            if violation.file_path not in file_violation_count:
                file_violation_count[violation.file_path] = 0
            file_violation_count[violation.file_path] += 1
            
            # Añadir a lista de violaciones
            report["violations"].append({
                "file_path": violation.file_path,
                "line_number": violation.line_number,
                "violation_type": violation.violation_type,
                "pattern_matched": "***REDACTED***",  # No mostrar datos reales
                "confidence": violation.confidence,
                "severity": violation.severity,
                "timestamp": violation.timestamp.isoformat()
            })
        
        # Identificar archivos de alto riesgo (>5 violaciones)
        report["high_risk_files"] = [
            {"file": file, "violation_count": count}
            for file, count in file_violation_count.items()
            if count > 5
        ]
        
        return report

# CLI para ejecución
def main():
    import argparse
    
    parser = argparse.ArgumentParser(description='DLP Scanner for sensitive data detection')
    parser.add_argument('path', help='File or directory to scan')
    parser.add_argument('--output', '-o', help='Output file for report')
    parser.add_argument('--format', choices=['json', 'csv'], default='json', help='Output format')
    
    args = parser.parse_args()
    
    scanner = DLPScanner()
    
    if os.path.isfile(args.path):
        violations = scanner.scan_file(args.path)
    elif os.path.isdir(args.path):
        violations = scanner.scan_directory(args.path)
    else:
        print(f"Error: {args.path} is not a valid file or directory")
        return
    
    report = scanner.generate_report(violations)
    
    if args.output:
        with open(args.output, 'w') as f:
            if args.format == 'json':
                json.dump(report, f, indent=2)
            elif args.format == 'csv':
                # Implementar exportación CSV si es necesario
                pass
    else:
        print(json.dumps(report, indent=2))
    
    print(f"\nScan completed. Found {len(violations)} potential violations.")
    if report["violations_by_severity"].get("critical", 0) > 0:
        print(f"⚠️  {report['violations_by_severity']['critical']} CRITICAL violations found!")

if __name__ == "__main__":
    main()
```

---

## 📊 Dashboard de Compliance

### 1. Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "id": null,
    "title": "Compliance Dashboard",
    "tags": ["compliance", "audit", "sox", "pci-dss"],
    "timezone": "browser",
    "refresh": "30s",
    "panels": [
      {
        "id": 1,
        "title": "Compliance Score Overview",
        "type": "stat",
        "targets": [
          {
            "expr": "compliance_score_total",
            "legendFormat": "Overall Score"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 70},
                {"color": "green", "value": 90}
              ]
            },
            "unit": "percent"
          }
        }
      },
      {
        "id": 2,
        "title": "SOD Violations by Department",
        "type": "piechart",
        "targets": [
          {
            "expr": "sum by (department) (sox_sod_violations_total)",
            "legendFormat": "{{department}}"
          }
        ]
      },
      {
        "id": 3,
        "title": "Audit Events Timeline",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(audit_events_total[5m])",
            "legendFormat": "Audit Events/sec"
          }
        ]
      },
      {
        "id": 4,
        "title": "Failed Control Tests",
        "type": "table",
        "targets": [
          {
            "expr": "control_test_failures_total",
            "format": "table",
            "instant": true
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "excludeByName": {"Time": true},
              "indexByName": {},
              "renameByName": {
                "control_id": "Control ID",
                "control_name": "Control Name",
                "failure_reason": "Failure Reason",
                "owner": "Owner"
              }
            }
          }
        ]
      },
      {
        "id": 5,
        "title": "PCI-DSS Compliance Status",
        "type": "bargauge",
        "targets": [
          {
            "expr": "pci_dss_requirement_compliance_percentage",
            "legendFormat": "Requirement {{requirement}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 80},
                {"color": "green", "value": 95}
              ]
            },
            "unit": "percent"
          }
        }
      },
      {
        "id": 6,
        "title": "Access Review Status",
        "type": "stat",
        "targets": [
          {
            "expr": "access_reviews_overdue_total",
            "legendFormat": "Overdue Reviews"
          },
          {
            "expr": "access_reviews_completed_this_quarter_total",
            "legendFormat": "Completed This Quarter"
          }
        ]
      },
      {
        "id": 7,
        "title": "DLP Violations by Severity",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum by (severity) (rate(dlp_violations_total[1h]))",
            "legendFormat": "{{severity}}"
          }
        ]
      },
      {
        "id": 8,
        "title": "Change Management Metrics",
        "type": "stat",
        "targets": [
          {
            "expr": "changes_without_approval_total",
            "legendFormat": "Unauthorized Changes"
          },
          {
            "expr": "emergency_changes_pending_review_total", 
            "legendFormat": "Emergency Changes Pending Review"
          }
        ]
      }
    ],
    "time": {
      "from": "now-24h",
      "to": "now"
    }
  }
}
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Implementar Controles SOX

**Objetivo**: Configurar controles SOX automatizados.

**Tareas**:

1. Implementar verificación SOD
2. Configurar controles ITGC
3. Crear reportes de cumplimiento
4. Configurar alertas de violaciones

### Ejercicio 2: Configurar Monitoreo PCI-DSS

**Objetivo**: Implementar controles PCI-DSS.

**Componentes**:

- Falco rules para detección de acceso a datos de tarjetas
- DLP scanner para detectar números de tarjetas
- Monitoreo de acceso a sistemas de pago
- Alertas de seguridad en tiempo real

### Ejercicio 3: Dashboard de Compliance

**Objetivo**: Crear dashboard centralizado de compliance.

**Tareas**:

- Configurar métricas de compliance
- Crear visualizaciones en Grafana
- Configurar alertas automáticas
- Implementar reportes periódicos

---

## 🧪 Laboratorio

### Lab 1: Stack Completo de Compliance

1. **Desplegar infraestructura**:

   ```bash
   docker-compose -f docker-compose.compliance.yml up -d
   ```

2. **Configurar controles**:
   - SOX controls con Python
   - PCI-DSS monitoring con Falco
   - DLP scanning en repositorios

3. **Configurar reportes**:
   - Dashboard de Grafana
   - Alertas automatizadas
   - Reportes periódicos

### Lab 2: Auditoría Automatizada

1. **Implementar controles automatizados**
2. **Configurar evidencia automática**
3. **Probar efectividad de controles**
4. **Generar reportes de auditoría**

---

## 📖 Best Practices

### 1. Principios de Compliance

```yaml
compliance_principles:
  automation:
    - "Automatizar controles donde sea posible"
    - "Reducir dependencia de procesos manuales"
    - "Validación continua vs. punto en el tiempo"
  
  documentation:
    - "Documentar todos los controles y procesos"
    - "Mantener evidencia de funcionamiento"
    - "Trazabilidad completa de cambios"
  
  monitoring:
    - "Monitoreo continuo de compliance"
    - "Alertas proactivas de desviaciones"
    - "Métricas de efectividad de controles"
```

### 2. Gestión de Evidencias

```yaml
evidence_management:
  collection:
    - "Recolección automatizada de evidencias"
    - "Timestamps y firmas digitales"
    - "Almacenamiento inmutable"
  
  retention:
    - "Políticas de retención por regulación"
    - "Backup y recuperación de evidencias"
    - "Acceso controlado y auditado"
  
  presentation:
    - "Reportes automatizados para auditores"
    - "Formatos estándar y legibles"
    - "Acceso self-service para stakeholders"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Arquitectura de compliance** con stack de auditoría completo
2. **Controles SOX** - Segregación de funciones e ITGC
3. **Monitoreo PCI-DSS** - Protección de datos de tarjetas
4. **Data Loss Prevention** - Detección de datos sensibles
5. **Auditoría automatizada** - Controles y evidencias automáticas
6. **Dashboard de compliance** - Visibilidad centralizada
7. **Reportes automáticos** - Cumplimiento continuo
8. **Best practices** - Principios de compliance en DevOps

### Próximas etapas

- **Módulo 09**: Vulnerability scanning avanzado
- **Módulo 10**: Distributed tracing con Jaeger
- **Módulo 11**: Application Performance Monitoring

¡Continúa con el siguiente módulo para dominar vulnerability scanning!
