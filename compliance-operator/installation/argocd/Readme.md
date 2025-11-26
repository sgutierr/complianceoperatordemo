# ArgoCD Applications para Compliance Operator

Este directorio contiene las aplicaciones de ArgoCD para instalar el Compliance Operator y sus ejemplos.

## Archivos

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

### 2. Aplicar las aplicaciones

Las aplicaciones están configuradas con sync waves para asegurar el orden correcto:
- **Wave 0**: Instalación del operador (compliance-operator-installation)
- **Wave 1**: Ejemplos y configuraciones (compliance-operator-examples)

```bash
# Aplicar la aplicación de instalación
oc apply -f compliance-operator-installation.yaml

# Aplicar la aplicación de ejemplos
oc apply -f compliance-operator-examples.yaml
```

O aplicar ambas a la vez:

```bash
oc apply -f .
```

## Características

### compliance-operator-installation
- Crea el namespace `openshift-compliance` si no existe
- Instala el OperatorGroup y Subscription para el Compliance Operator
- Sincronización automática habilitada
- Auto-healing y pruning habilitados

### compliance-operator-examples
- Sincroniza todos los ejemplos de los directorios `demo1` y `demo2`
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

## Notas

- Las aplicaciones están configuradas para sincronización automática
- El namespace `openshift-compliance` se creará automáticamente si no existe
- Los recursos se eliminarán automáticamente si se eliminan del repositorio (prune: true)
- Las aplicaciones tienen retry configurado en caso de errores temporales

