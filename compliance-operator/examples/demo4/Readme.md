# Demo 4 - CIS Benchmark Compliance Scan

Este demo muestra cómo ejecutar un escaneo de cumplimiento contra el CIS Red Hat OpenShift Container Platform 4 Benchmark.

## Prerrequisitos

- OpenShift cluster
- Client host configurado (ver [Client host setup](../../installation/Readme.md))
- OpenShift CLI tools instalados en el client host
- EBS como backend de almacenamiento para soportar un volumen persistente de 1GB como está configurado en el ScanSetting del Compliance Operator

> **Nota**: Con el método seguido en esta guía no necesitas un PV que soporte ReadWriteMany.

## Escenario

Simularemos un escenario donde queremos verificar OpenShift contra el **CIS Red Hat OpenShift Container Platform 4 Benchmark** que puede descargarse gratuitamente del sitio web de CIS (https://www.cisecurity.org/cis-benchmarks/) filtrando por "Server Software" y haciendo clic en "Expand to see related content" bajo Kubernetes.

## Pasos

### 1. Crear el ScanSettingBinding

Ejecuta el siguiente comando para crear un binding que ejecute el escaneo CIS:

```bash
oc compliance bind -N ocp4-cis-binding profile/ocp4-cis profile/ocp4-cis-node -n openshift-compliance
```

### 2. Monitorear el escaneo

Observa el progreso del escaneo:

```bash
oc get compliancescan -w -n openshift-compliance
```

### 3. Ver los resultados

Una vez completado el escaneo, puedes ver los resultados:

```bash
oc get ComplianceCheckResult -n openshift-compliance
```

### 4. Ver las remediaciones

Para ver las remediaciones disponibles:

```bash
oc get ComplianceRemediation -n openshift-compliance
```

### 5. Descargar resultados en bruto

Descarga los resultados en bruto para generar reportes:

```bash
mkdir -p ~/compliance-results
oc compliance fetch-raw scansettingbindings ocp4-cis-binding -o ~/compliance-results/ -n openshift-compliance
```

### 6. Descomprimir resultados

Descomprime los resultados comprimidos:

```bash
bunzip2 -c <archivo_comprimido> > <archivo_xml>
```

### 7. Generar reporte HTML

Genera un reporte HTML usando OpenSCAP:

```bash
oscap xccdf generate report <archivo_xml> > <reporte_html>
```

## Referencias

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [OpenShift Compliance Operator Documentation](https://docs.openshift.com/container-platform/latest/security/compliance_operator/compliance-operator-understanding.html)
