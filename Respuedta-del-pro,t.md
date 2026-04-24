¡Hola! Soy tu colaborador IA. Es un placer ayudarte a estructurar este proyecto ambicioso. Vamos a configurar `proyectoperfumeria` siguiendo una secuencia lógica y profesional para asegurar que tu entorno esté listo y tu código sea escalable.

---

## 1. Verificación y Preparación del Entorno

Antes de tocar el código, asegurémonos de que las herramientas base están operativas. Abre tu terminal (VS Code o terminal del sistema) y ejecuta:

```bash
# 1. Verificar Flutter
flutter --version

# 2. Verificar Dart
dart --version

# 3. Verificar Firebase CLI
firebase --version

# 4. Instalar FlutterFire CLI (si no lo tienes)
dart pub global activate flutterfire_cli
```

*Si alguno falla, instálalo desde los sitios oficiales (flutter.dev y firebase.google.com).*

---

## 2. Configuración de la Estructura de Agente (`.agents`)

Vamos a crear la estructura de gestión de tus habilidades de IA. Esta carpeta servirá como el "cerebro" de tu automatización.

Crea la carpeta en la raíz de tu proyecto `proyectoperfumeria/`:

```bash
mkdir .agents scripts ejemplos resources
```

### Creando `SKILL.md`
Crea el archivo `.agents/SKILL.md` con el siguiente contenido para definir tus capacidades:

```markdown
# Habilidad Agente: Perfumeria Automator
## Skills Definidos:
- **Design Skill**: Generación de layouts UI, paleta de colores y componentes visuales.
- **Code Skill**: Gestión de CRUD, integración Firestore y lógica de negocio (Dart/Flutter).
- **Scraping Skill**: Extracción de datos de proveedores de perfumes para la DB.

## Estado
- Entorno: Flutter/Firebase configurado.
- Objetivo: App móvil Perfumería (CRUD).
```

---

## 3. Conectividad Firebase y Configuración

Para conectar tu app a Firestore:

1.  En la consola de Firebase, crea un proyecto llamado `proyectoperfumeria`.
2.  Registra tu app (Android/Flutter).
3.  En tu terminal, dentro de la carpeta `proyectoperfumeria`, ejecuta:
    ```bash
    firebase login
    flutterfire configure
    ```
    *Selecciona tu proyecto y plataformas (Android/iOS).*

4.  Asegúrate de tener las dependencias en `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
```
Luego ejecuta: `flutter pub get`.

---

## 4. Estructura del Proyecto (`lib/`)

Organiza tus archivos para mantener el orden. Crea estas subcarpetas dentro de `lib/`:
`models/`, `services/`, `screens/`.

### Modelo de Datos (`lib/models/perfume.dart`)

```dart
class Perfume {
  final String id;
  final String nombre;
  final double precio;

  Perfume({required this.id, required this.nombre, required this.precio});

  factory Perfume.fromFirestore(Map<String, dynamic> data, String id) {
    return Perfume(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
    );
  }

  Map<String, dynamic> toMap() => {'nombre': nombre, 'precio': precio};
}
```

### Servicio CRUD (`lib/services/perfume_service.dart`)

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/perfume.dart';

class PerfumeService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('perfumes');

  Future<void> addPerfume(String nombre, double precio) async => 
      await _db.add({'nombre': nombre, 'precio': precio});

  Future<void> updatePerfume(String id, String nombre, double precio) async =>
      await _db.doc(id).update({'nombre': nombre, 'precio': precio});

  Future<void> deletePerfume(String id) async => await _db.doc(id).delete();
}
```

---

## 5. UI: Pantalla Principal (`lib/screens/home_screen.dart`)

Esta es la base para tu navegación y visualización de datos.

```dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../services/perfume_service.dart';

class HomeScreen extends StatelessWidget {
  final PerfumeService _service = PerfumeService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Perfumería App")),
      body: StreamBuilder(
        stream: FirebaseFirestore.instance.collection('perfumes').snapshots(),
        builder: (context, AsyncSnapshot<QuerySnapshot> snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          return ListView(
            children: snapshot.data!.docs.map((doc) {
              return ListTile(
                title: Text(doc['nombre']),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () => _service.deletePerfume(doc.id),
                ),
              );
            }).toList(),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _service.addPerfume("Nuevo Perfume", 99.99),
      ),
    );
  }
}
```

---

## 6. Integración en `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MaterialApp(home: HomeScreen()));
}
```

---

Has establecido una base sólida: la estructura de agentes, la configuración de Firebase y el CRUD funcional.

¿Te gustaría que procedamos a crear un **script de automatización** en la carpeta `scripts/` que genere automáticamente los modelos de datos cuando cambies los campos de la base de datos?
