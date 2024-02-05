# DO NOT MERGE

This file is a todo list for the implementation of integration tests for the rook/ceph/swift with keystone functionality.

## github-Test-Thema anschauen (jan)
- Warum laufen die Tests nicht mehr durch (und werden als gecanceled angezeigt)
- https://github.com/jklippel/rook/pull/1/commits/14105bef7e76afd198158ac21935bf60270011e3 hat das Problem gelöst (cherry-picked von upstream/master)
- Nach einigen guten Durchläufen laufen die Tests jetzt wieder nicht mehr.

## (done) Refactor: installHelmChart (ceph_base_keystone_test.go) (jan) 
- Refactor: Helm-Chart Installations-Funktion installHelmChart wurde um trust-manager spezifische Sachen erweitert, das
  sollte nicht so sein.

## Was wollen wir testen?

### Swift 

Das gibts alles noch nicht, bin mir aber nicht sicher, wie wir das außerhalb des Clusters testen wollen.
Dann bräuchte es noch einen Ingress-Weg zum Keystone bzw. einen von extern erreichbaren Service.

- (implementiert) Container anlegen
- (implementiert) Datei in Container ablegen
- (implementiert) Datei aus Container laden
- (implementiert) Datei in Container löschen
- (implementiert) Container löschen
- (implementiert) unberechtigter Zugriff (jan)
- berechtigte Zugriffe (Read-only, Write-only(?), Admin?) (jan)

### S3 (silvio)

Wie testen wir das? (mit welchem S3-Client?, kann man das mit openstack-cli machen?)

- Bucket anlegen
- Bucket verwenden
- Bucket löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)

## S3 Credentials in Keystone anlegen anstelle des ObjectStoreUsers aus den normalen Tests (silvio)
-> Access keys aus Keystone in das jetzige Secret legen(?)

## CephObjektStoreUser + Subuser (silvio)

- Was machen wir mit der Funktionalität?
- (done) Trennen wir den MR/PR/Branch auf? --> ja
- (done) Testen wir diese Funktionalität auch? --> in einem weiteren PR

## Spezifikation lesen und schauen ob wir alles abgedeckt haben (jan/silvio)

ich (jan) finde es sieht ganz gut aus, allerdings sind da natürlich die Subuser Sachen beschrieben. Aber ich denke
dass man nicht alles in einem PR implementieren muss.

## nach erstem rudimentären mal den PR aufmachen und die Community einbinden

## GetKeystoneUserSecret anpassen (jan)

In object.go die Funktion GetKeystoneUserSecret anpassen/variabler machen

## Refaktor: ceph_base_keystone_test.go credentials in Konstanten auslagern (jan)

##  (low-prio) Nebenschauplatz: tests/scripts/generate-tls-config.sh kubectl version --short schlägt fehl, denn es ist nicht mehr aktuell

## (low-prio) Ceph-Swift-Dokumentation anschauen und verbessern

## (low-prio) einen Blick auf Coverage werfen

## (low-prio) Schauen, warum das ceph_manifest.go Dinge verwenden möchte, die nicht im CRD sind

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

## (low-prio) Brauchen wir ein neueres Keystone als Yoga?

Das letzte Yaook-Image ist für Yoga, aber man könnte da ruhig auch ein paar neuere bauen.
(Ist mir beim Doku-Lesen aufgefallen :)

## COSI-Fehler / wo kommen die her?
failed to reconcile CephObjectStore "keystoneauth-ns/default". failed to create object store deployments: failed to get COSI user "cosi": Get "http://rook-ceph-rgw-default.keystoneauth-ns.svc:80/admin/user?format=json&uid=cosi": dial tcp 10.106.142.173:80: connect: connection refused

