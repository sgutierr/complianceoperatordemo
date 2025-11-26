# Demo 3 - Additional Security CustomRules

Este directorio contiene ejemplos adicionales de CustomRule para el Compliance Operator que cubren mejores prácticas de seguridad para contenedores y pods.

## CustomRules incluidos

### 1. **no-latest-image-tag.yaml**
Verifica que los pods no usen la etiqueta 'latest' para las imágenes de contenedores, lo que puede llevar a despliegues impredecibles.

### 2. **pods-must-have-resource-limits.yaml**
Asegura que todos los contenedores tengan límites de recursos (CPU y memoria) definidos para prevenir el agotamiento de recursos.

### 3. **pods-must-not-run-as-root.yaml**
Verifica que los contenedores no se ejecuten como usuario root (UID 0) para seguir el principio de menor privilegio.

### 4. **secrets-must-not-be-in-env-vars.yaml**
Asegura que los datos sensibles de Secrets no se expongan como variables de entorno en texto plano.

### 5. **critical-namespaces-must-have-networkpolicy.yaml**
Verifica que los namespaces críticos tengan al menos una NetworkPolicy definida para aplicar segmentación de red.

### 6. **pods-must-have-readiness-probe.yaml**
Asegura que todos los contenedores tengan readiness probes configurados para un enrutamiento adecuado del tráfico.

### 7. **pods-must-not-use-hostnetwork.yaml**
Verifica que los pods no usen hostNetwork, lo que puede exponer el pod a la pila de red del host.

### 8. **pods-must-not-use-privileged.yaml**
Asegura que los contenedores no se ejecuten en modo privilegiado, lo que otorga casi todas las capacidades de la máquina host.

### 9. **namespaces-must-have-pod-security-standards.yaml**
Verifica que los namespaces tengan etiquetas de Pod Security Standards configuradas.

## Uso

### Aplicar todos los CustomRules

```bash
oc apply -f compliance-operator/examples/demo3/
```

### Aplicar el TailoredProfile y ScanSettingBinding

```bash
# Aplicar el TailoredProfile
oc apply -f compliance-operator/examples/demo3/tailoredprofile-demo3.yaml

# Aplicar el ScanSettingBinding para ejecutar el escaneo
oc apply -f compliance-operator/examples/demo3/scansettingbinding-demo3.yaml
```

### Verificar el estado del escaneo

```bash
# Ver el estado del escaneo
oc get compliancescans -n openshift-compliance

# Ver los resultados
oc get compliancecheckresults -n openshift-compliance
```

## Referencias

- [OpenShift Compliance Operator Documentation](https://docs.openshift.com/container-platform/latest/security/compliance_operator/compliance-operator-understanding.html)
- [Common Expression Language (CEL)](https://github.com/google/cel-spec)

