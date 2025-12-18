# Compliance Operator Demo

Este repositorio contiene ejemplos y configuraciones para el Compliance Operator de OpenShift, incluyendo instalación mediante ArgoCD y múltiples demos con CustomRules personalizados para diferentes escenarios de seguridad y cumplimiento.

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Estructura del Repositorio](#estructura-del-repositorio)
- [Instalación](#instalación)
- [Ejemplos y Demos](#ejemplos-y-demos)
- [Referencias](#referencias)

## Descripción General

El Compliance Operator de OpenShift permite automatizar el escaneo de cumplimiento y la remediación de problemas de seguridad en clústeres de Kubernetes/OpenShift. Este repositorio proporciona:

- **Instalación automatizada** mediante ArgoCD
- **Ejemplos prácticos** de CustomRules para diferentes escenarios de seguridad
- **Configuraciones listas para usar** de TailoredProfiles y ScanSettingBindings
- **Demos progresivos** desde conceptos básicos hasta escaneos CIS completos

## Estructura del Repositorio

```
compliance-operator/
├── examples/              # Ejemplos de CustomRules y configuraciones
│   ├── demo1/            # Security Checks Básicos
│   ├── demo2/            # NetworkPolicy Security Checks
│   ├── demo3/            # Container and Pod Security Best Practices
│   └── demo4/            # CIS Benchmark Compliance Scan
└── installation/         # Scripts y configuraciones de instalación
    ├── argocd/           # Aplicaciones de ArgoCD para instalación automatizada
    └── operator/         # Instalación manual del operador
```

## Instalación

### Opción 1: Instalación mediante ArgoCD (Recomendado)

La instalación mediante ArgoCD permite gestionar el Compliance Operator y sus ejemplos de forma declarativa y automatizada.

#### Prerrequisitos

- ArgoCD instalado en el clúster
- Permisos para crear recursos en el namespace `openshift-compliance`
- Acceso al repositorio Git donde se encuentra este código

#### Pasos de Instalación

1. **Configurar permisos RBAC** (IMPORTANTE - debe aplicarse primero):

```bash
oc apply -f compliance-operator/installation/argocd/argocd-rbac.yaml
```

2. **Aplicar las aplicaciones de ArgoCD**:

```bash
# Aplicar aplicación de instalación del operador
oc apply -f compliance-operator/installation/argocd/compliance-operator-installation.yaml

# Aplicar aplicación de ejemplos
oc apply -f compliance-operator/installation/argocd/compliance-operator-examples.yaml
```

3. **Verificar la instalación**:

```bash
# Verificar que el operador esté instalado
oc get csv -n openshift-compliance

# Verificar pods del operador
oc get pods -n openshift-compliance

# Verificar aplicaciones de ArgoCD
oc get applications -n argocd
```

Para más detalles, consulta el [README de ArgoCD](compliance-operator/installation/argocd/Readme.md).

### Opción 2: Instalación Manual

Para instalar el operador manualmente:

```bash
# Crear namespace
oc apply -f compliance-operator/installation/operator/namespace.yaml

# Crear subscription
oc apply -f compliance-operator/installation/operator/susbcription.yaml
```

### Configuración del Cliente (oc-compliance plugin)

Para usar las utilidades del Compliance Operator desde tu máquina cliente, necesitas instalar el plugin `oc-compliance`:

```bash
# Instalar podman (si no está disponible)
sudo yum -y install podman

# Configurar autenticación con Red Hat registry
export REGISTRY_AUTH_PATH=~
mkdir -p $REGISTRY_AUTH_PATH/containers
oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json

# Descargar e instalar el plugin
export OC_PATH=$(whereis -b oc | awk '{ print $2 }' | sed 's/\/oc//g')
podman run --authfile $REGISTRY_AUTH_PATH/containers/rh-pull-secret.json --rm --entrypoint /bin/cat registry.redhat.io/compliance/oc-compliance-rhel8 /usr/bin/oc-compliance > /tmp/oc-compliance && chmod +x /tmp/oc-compliance
mv /tmp/oc-compliance $OC_PATH/oc-compliance

# Instalar herramientas adicionales para reportes
sudo yum -y install openscap-scanner tree bzip2

# Verificar instalación
oc compliance --help
```

Para más detalles, consulta el [README de instalación](compliance-operator/installation/Readme.md).

## Ejemplos y Demos

### Demo 1: Security Checks Básicos

Este demo incluye verificaciones de seguridad básicas para administración del clúster, configuración de registros y detección de bases de datos no aprobadas.

**CustomRules incluidos:**
- `cluster-admin-allow-list.yaml`: Audita el acceso cluster-admin contra una lista de permitidos (alineado con CIS 5.1.1)
- `allowed-registries-configured.yaml`: Verifica que los registros confiables estén definidos en la configuración de imágenes del clúster
- `disallow-shadow-databases.yaml`: Previene que los equipos de aplicación desplieguen instancias de bases de datos no aprobadas

**Aplicar el demo:**

```bash
oc apply -f compliance-operator/examples/demo1/
oc apply -f compliance-operator/examples/demo1/custom-security-scan.yaml
```

### Demo 2: NetworkPolicy Security Checks

Este demo se enfoca en la seguridad de red mediante NetworkPolicies.

**CustomRules incluidos:**
- `netpol-disallow-allow-all-in-labeled-namespaces.yaml`: Detecta NetworkPolicies que permiten todo el tráfico en namespaces etiquetados
- `netpol-require-deny-all-in-labeled-namespaces.yaml`: Requiere una NetworkPolicy que deniegue todo el tráfico en namespaces etiquetados

**Aplicar el demo:**

```bash
oc apply -f compliance-operator/examples/demo2/
oc apply -f compliance-operator/examples/demo2/ScanSettingBinding.yaml
```

### Demo 3: Container and Pod Security Best Practices

Este demo contiene ejemplos adicionales de CustomRules que cubren mejores prácticas de seguridad para contenedores y pods.

**CustomRules incluidos:**
- `no-latest-image-tag.yaml`: Verifica que los pods no usen la etiqueta 'latest' para imágenes
- `pods-must-have-resource-limits.yaml`: Asegura que todos los contenedores tengan límites de recursos definidos
- `pods-must-not-run-as-root.yaml`: Verifica que los contenedores no se ejecuten como root (UID 0)
- `sec-must-not-be-in-env-vars.yaml`: Asegura que los datos sensibles de Secrets no se expongan como variables de entorno
- `critical-namespaces-must-have-networkpolicy.yaml`: Verifica que los namespaces críticos tengan NetworkPolicy definida
- `pods-must-have-readiness-probe.yaml`: Asegura que todos los contenedores tengan readiness probes configurados
- `pods-must-not-use-hostnetwork.yaml`: Verifica que los pods no usen hostNetwork
- `pods-must-not-use-privileged.yaml`: Asegura que los contenedores no se ejecuten en modo privilegiado
- `namespaces-must-have-pod-security-standards.yaml`: Verifica que los namespaces tengan etiquetas de Pod Security Standards

**Aplicar el demo:**

```bash
# Aplicar todos los CustomRules
oc apply -f compliance-operator/examples/demo3/

# Aplicar el TailoredProfile y ScanSettingBinding
oc apply -f compliance-operator/examples/demo3/tailoredprofile-demo3.yaml
oc apply -f compliance-operator/examples/demo3/scansettingbinding-demo3.yaml

# Verificar el estado del escaneo
oc get compliancescans -n openshift-compliance
oc get compliancecheckresults -n openshift-compliance
```

Para más detalles, consulta el [README de Demo 3](compliance-operator/examples/demo3/Readme.md).

### Demo 4: CIS Benchmark Compliance Scan

Este demo muestra cómo ejecutar un escaneo de cumplimiento contra el **CIS Red Hat OpenShift Container Platform 4 Benchmark**.

**Prerrequisitos:**
- OpenShift cluster
- Client host configurado con oc-compliance plugin
- EBS como backend de almacenamiento para soportar un volumen persistente de 1GB

**Aplicar el demo:**

```bash
# Crear el ScanSettingBinding
oc compliance bind -N ocp4-cis-binding profile/ocp4-cis profile/ocp4-cis-node -n openshift-compliance

# Monitorear el escaneo
oc get compliancescan -w -n openshift-compliance

# Ver los resultados
oc get ComplianceCheckResult -n openshift-compliance

# Ver las remediaciones
oc get ComplianceRemediation -n openshift-compliance

# Descargar resultados en bruto
mkdir -p ~/compliance-results
oc compliance fetch-raw scansettingbindings ocp4-cis-binding -o ~/compliance-results/ -n openshift-compliance
```

Para más detalles, consulta el [README de Demo 4](compliance-operator/examples/demo4/Readme.md).

## Verificación y Monitoreo

### Verificar estado de escaneos

```bash
# Ver escaneos en ejecución
oc get compliancescans -n openshift-compliance

# Ver resultados de verificaciones
oc get compliancecheckresults -n openshift-compliance

# Ver remediaciones disponibles
oc get complianceremediations -n openshift-compliance
```

### Ver detalles de un resultado específico

```bash
oc compliance view-result <nombre-del-resultado> -n openshift-compliance
```

### Descargar resultados en bruto

```bash
oc compliance fetch-raw scansettingbindings <nombre-binding> -o <directorio-salida> -n openshift-compliance
```

## Referencias

- [OpenShift Compliance Operator Documentation](https://docs.openshift.com/container-platform/latest/security/compliance_operator/compliance-operator-understanding.html)
- [Compliance Operator Customization Guide](https://ralvares.github.io/openshift-security-framework/docs/html/compliance-operator-customization.html)
- [Common Expression Language (CEL)](https://github.com/google/cel-spec)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [OpenSCAP Documentation](https://www.open-scap.org/)

## Contribuciones

Este repositorio contiene ejemplos y configuraciones para el Compliance Operator. Siéntete libre de adaptar estos ejemplos a tus necesidades específicas.

## Licencia

Este repositorio contiene ejemplos y configuraciones de referencia para el Compliance Operator de OpenShift.
