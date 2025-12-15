# ArgoCD Applications para Compliance Operator

Este directorio contiene las aplicaciones de ArgoCD para instalar el Compliance Operator y sus ejemplos.

## Archivos

- **argocd-rbac.yaml**: Role y RoleBinding para otorgar permisos a ArgoCD sobre recursos del Compliance Operator
- **argocd-rbac-application.yaml**: Aplicación de ArgoCD para gestionar los permisos RBAC (debe aplicarse primero)
- **compliance-operator-installation.yaml**: Aplicación para instalar el Compliance Operator (Namespace, OperatorGroup, Subscription)
- **compliance-operator-examples.yaml**: Aplicación para sincronizar los ejemplos y configuraciones del Compliance Operator

## Configuración

### 1. Actualizar el repositorio Git

Antes de aplicar las aplicaciones, actualiza la URL del repositorio en ambos archivos:

```yaml
source:
  repoURL: https://github.com/sgutierr/complianceoperatordemo.git  
  targetRevision: HEAD
```

### 2. Aplicar los permisos RBAC (IMPORTANTE)

**Antes de aplicar las aplicaciones de ArgoCD**, es necesario otorgar permisos al ServiceAccount de ArgoCD para que pueda crear recursos del Compliance Operator:

```bash
# Aplicar el RBAC directamente (recomendado para la primera vez)
oc apply -f argocd-rbac.yaml

# O aplicar la aplicación de ArgoCD para el RBAC
oc apply -f argocd-rbac-application.yaml
```

### 3. Aplicar las aplicaciones

Las aplicaciones están configuradas con sync waves para asegurar el orden correcto:
- **Wave -1**: Permisos RBAC (argocd-rbac-application) - debe aplicarse primero
- **Wave 0**: Instalación del operador (compliance-operator-installation)
- **Wave 1**: Ejemplos y configuraciones (compliance-operator-examples)

```bash
# Aplicar la aplicación de instalación
oc apply -f compliance-operator-installation.yaml

# Aplicar la aplicación de ejemplos
oc apply -f compliance-operator-examples.yaml
```

O aplicar todas a la vez:

```bash
oc apply -f .
```

## Características

### argocd-rbac
- Otorga permisos al ServiceAccount de ArgoCD para gestionar recursos del Compliance Operator
- Permisos sobre: CustomRule, TailoredProfile, ScanSettingBinding, ComplianceScan, etc.
- **Debe aplicarse antes** de las otras aplicaciones para evitar errores de permisos

### compliance-operator-installation
- Crea el namespace `openshift-compliance` si no existe
- Instala el OperatorGroup y Subscription para el Compliance Operator
- Sincronización automática habilitada
- Auto-healing y pruning habilitados

### compliance-operator-examples
- Sincroniza todos los ejemplos de los directorios `demo1`, `demo2`, `demo3`, y `demo4`
- Se sincroniza después de la instalación del operador (sync-wave: 1)
- Sincronización automática habilitada
- Auto-healing y pruning habilitados

## Verificación

Verificar el estado de las aplicaciones:

```bash
# Ver aplicaciones
oc get applications -n argocd

# Ver detalles de la instalación
oc get application compliance-operator-installation -n argocd -o yaml

# Ver detalles de los ejemplos
oc get application compliance-operator-examples -n argocd -o yaml

# Ver logs de sincronización
argocd app get compliance-operator-installation
argocd app get compliance-operator-examples
```

Verificar que el operador esté instalado:

```bash
# Verificar CSV (ClusterServiceVersion)
oc get csv -n openshift-compliance

# Verificar pods del operador
oc get pods -n openshift-compliance

# Verificar que los ejemplos se hayan aplicado
oc get profiles -n openshift-compliance
oc get scansettingbindings -n openshift-compliance
```

## Solución de problemas

### Error: "cannot create resource customrules/tailoredprofiles"

Si ves este error, significa que el ServiceAccount de ArgoCD no tiene permisos. Solución:

```bash
# Aplicar el RBAC
oc apply -f argocd-rbac.yaml

# Verificar que el RoleBinding se creó correctamente
oc get rolebinding argocd-compliance-operator-manager -n openshift-compliance

# Verificar que el Role tiene los permisos correctos
oc get role argocd-compliance-operator-manager -n openshift-compliance -o yaml
```

## Notas

- **IMPORTANTE**: Aplica el RBAC antes de las otras aplicaciones para evitar errores de permisos
- Las aplicaciones están configuradas para sincronización automática
- El namespace `openshift-compliance` se creará automáticamente si no existe
- Los recursos se eliminarán automáticamente si se eliminan del repositorio (prune: true)
- Las aplicaciones tienen retry configurado en caso de errores temporales

