## Lab10_6: Detección de accesos anómalos con IA en Ubuntu

Objetivo: utilizar el algoritmo Isolation Forest de Machine Learning para identificar comportamientos sospechosos relacionados con el acceso a archivos.

### 1. Preparar el entorno desde la terminal

```
mkdir ~/lab-ia
cd ~/lab-ia
sudo apt install python3-venv -y
python3 -m venv .venv
source .venv/bin/activate
pip install scikit-learn
```

### 2. Crear el programa

```
nano detector.py
```

Pegar el siguiente código:

```python
from sklearn.ensemble import IsolationForest# [Hora de acceso, cantidad de archivos]accesos = [    [8, 3],    [9, 4],    [10, 5],    [11, 3],    [14, 6],    [15, 4],    [16, 5],    [3, 150]]modelo = IsolationForest(contamination=0.125, random_state=42)modelo.fit(accesos)for acceso, resultado in zip(accesos, modelo.predict(accesos)):    estado = "SOSPECHOSO" if resultado == -1 else "NORMAL"    print(f"Hora: {acceso[0]} | Archivos: {acceso[1]} | {estado}")
```

Guardar con `Ctrl + O`, `Enter` y `Ctrl + X`.

### 3. Ejecutar el modelo

```
python3 detector.py
```

Resultado esperado:

```
Hora: 8  | Archivos: 3   | NORMAL
Hora: 9  | Archivos: 4   | NORMAL
Hora: 10 | Archivos: 5   | NORMAL
Hora: 11 | Archivos: 3   | NORMAL
Hora: 14 | Archivos: 6   | NORMAL
Hora: 15 | Archivos: 4   | NORMAL
Hora: 16 | Archivos: 5   | NORMAL
Hora: 3  | Archivos: 150 | SOSPECHOSO
```

### 4. ¿Cómo funciona Isolation Forest?

El algoritmo identifica observaciones que se pueden aislar más fácilmente mediante divisiones aleatorias de los datos.

En nuestro ejemplo, acceder a 150 archivos a las 03:00 es muy diferente del comportamiento habitual, por lo que el modelo puede clasificarlo como una anomalía.

- `1`: comportamiento normal.
- `-1`: comportamiento anómalo.

Importante: estamos trabajando con accesos simulados. El modelo no consulta los permisos reales de Linux ni bloquea usuarios.

### Individual Challenge: AI-Based Access Anomaly Detection

El estudiante deberá implementar un modelo de detección de anomalías mediante **Isolation Forest** en Ubuntu Linux, utilizando Python y scikit-learn desde la terminal. Para ello, generará un conjunto de al menos **20 registros simulados de acceso a archivos**, considerando la hora de acceso y la cantidad de documentos consultados, e incorporará algunos comportamientos inusuales. Finalmente, deberá ejecutar el modelo, identificar los accesos clasificados como sospechosos y presentar un informe individual con el código utilizado, capturas de los resultados y una breve explicación sobre cómo la Inteligencia Artificial puede complementar los mecanismos tradicionales de control de acceso de un sistema operativo.
