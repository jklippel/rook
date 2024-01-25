# DO NOT MERGE

This file is a todo list for the implementation of integration tests for the rook/ceph/swift with keystone functionality.

## (done) Keystone-Deployment hinzufügen

## (done) Schauen, warum das cephfs getestet wird

Ich denke das läuft halt immer beim Aufsetzen mit.

## Was wollen wir testen?

### Swift

Das gibts alles noch nicht, bin mir aber nicht sicher, wie wir das außerhalb des Clusters testen wollen.
Dann bräuchte es noch einen Ingress-Weg zum Keystone bzw. einen von extern erreichbaren Service.

- (implementiert) Container anlegen
- (implementiert) Datei in Container ablegen
- (implementiert) Datei aus Container laden
- Datei in Container löschen
- (implementiert) Container löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)
- Smoke-Test? (many requests to keystone)

### S3

- Bucket anlegen
- Bucket verwenden
- Bucket löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)

## CephObjektStoreUser + Subuser

- Was machen wir mit der Funktionalität?
- Trennen wir den MR/PR/Branch auf?
- Testen wir diese Funktionalität auch?

## Spezifikation lesen und schauen ob wir alles abgedeckt haben

## nach erstem rudimentären mal den PR aufmachen und die Community einbinden

### GetKeystoneUserSecret anpassen

In object.go die Funktion GetKeystoneUserSecret anpassen/variabler machen

## S3 Credentials in Keystone anlegen anstelle des ObjectStoreUsers aus den normalen Tests
-> Access keys aus Keystone in das jetzige Secret legen(?)

##  (low-prio) Nebenschauplatz: tests/scripts/generate-tls-config.sh kubectl version --short schlägt fehl, denn es ist nicht mehr aktuell

## (low-prio) Ceph-Swift-Dokumentation anschauen und verbessern

## (low-prio) einen Blick auf Coverage werfen

##  (low-prio) Schauen, warum das ceph_manifest.go Dinge verwenden möchte, die nicht im CRD sind

```
diff --git a/tests/framework/installer/ceph_manifests.go b/tests/framework/installer/ceph_manifests.go
index 5cba689e2..21aa32ee8 100644
--- a/tests/framework/installer/ceph_manifests.go
+++ b/tests/framework/installer/ceph_manifests.go
@@ -498,14 +498,9 @@ spec:
   {{ end }}
   gateway:
     resources: null
-    type: s3
     {{ if .TLS }}securePort: {{ .Port }}{{ else }}port: {{ .Port }}{{ end }}
     instances: {{ .ReplicaCount }}
-    {{ if .TLS }}sslCertificateRef: {{ .Name }}{{ end }}
-  healthCheck:
-    bucket:
-      disabled: false
-      interval: {{ if .TLS }}10s{{ else }}5s{{ end }}`
+    {{ if .TLS }}sslCertificateRef: {{ .Name }}{{ end }}`

        return renderTemplate(tmpl, spec)
 }

```
