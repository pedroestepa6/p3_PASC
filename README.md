# Práctica 3: Ecualización de una Señal QAM-16 Transmitida por Canal Multitrayectoria

## **1. Fundamentos Teóricos**

### **1.1 Constelación QAM-16 y Codificación Gray**
- **Modulación QAM-16**:  
  - Representa 4 bits por símbolo mediante amplitud y fase.  
  - Constelación de 16 puntos en cuadrícula 4x4 (valores I/Q de ±1, ±3).  
  - **Codificación Gray**: Símbolos adyacentes difieren en 1 bit para minimizar errores (ej: `0000` → `0001`).

### **1.2 Preparación de la Señal de Audio**
- **Fuente**: Audio telefónico (8 bits, 8 kHz, 64 kbps).  
- **Procesamiento**:  
  1. **Offset +128**: Convierte rango de [-128, 127] a [0, 255].  
  2. **División de bits**: Cada muestra se divide en:  
     - **4 LSB**: Transmitidos primero.  
     - **4 MSB**: Transmitidos después.  
  3. **Mapeo a QAM-16**: Usa `qammod` con codificación Gray.

### **1.3 Modelo del Canal**
- **Multitrayectoria**: Causa interferencia intersimbólica (ISI).  
- **Filtrado**:  
  - **Transmisor**: Pulso en coseno elevado (`p(t)`).  
  - **Receptor**: Filtro adaptado `p(-t)`.  
- **Respuesta del canal**: \( h(t) = q(t) * p(-t) \).  
- **Ruido AWGN**: SNR = 22 dB por bit (\(E_b/N_0\)).

---

## **2. Realización Práctica**

### **2.1 Caso 1: Ecualizador Fijo TSE-LS**
**Objetivo**: Diseñar ecualizador FIR de 9 coeficientes con mínimos cuadrados (LS).  

**Pasos**:  
1. **Carga de datos** (`Tx_fijo_variables.mat`):  
   - `x`: Audio original.  
   - `trainseq`: 300 símbolos de entrenamiento.  
   - `z`: Señal recibida (muestreada a 16 kHz).  

2. **Diseño del ecualizador**:  
   - **Matriz \(Z\)**: Muestras desplazadas de `z` (ej: \(Z[i,j] = z[i-j+\text{ini}+u]\)).  
   - **Solución LS**: \( \mathbf{f} = (Z^H Z)^{-1} Z^H \cdot \mathbf{trainseq} \).  
   - **Retardo \(u\)**: Optimizado (ej: \(u=2\)).  

3. **Evaluación**:  
   - **SNR**: \( 10 \log_{10} \left( \frac{\sum x^2}{\sum (x_{\text{sint}} - x)^2} \right) \).  
   - **SER**: Porcentaje de símbolos erróneos.  

---

### **2.2 Caso 2: Ecualizador Adaptable NLMS**
**Objetivo**: Manejar canales variables con LS inicial + NLMS.  

**Diferencias clave**:  
1. **Inicialización con LS**: Usando `trainseq`.  
2. **Adaptación NLMS**:  
   - Actualización:  
     \[
     \mathbf{f}_{\text{new}} = \mathbf{f}_{\text{old}} + \frac{\mu}{\|\mathbf{z}_n\|^2} \cdot e[n] \cdot \mathbf{z}_n^*
     \]  
   - **Paso \(\mu\)**: Ej: 0.01.  
3. **Modo Decision-Directed**: Usa símbolos demodulados como referencia.  

**Ventajas**: Mejor adaptación a cambios del canal.  

---

## **3. Aspectos Clave**

### **3.1 Retardo \(u\)**
- **Propósito**: Compensar retardos del canal.  
- **Optimización**: Probar valores (0 a 4) y seleccionar mejor SNR/SER.  

### **3.2 Funciones MATLAB**
- **`audio8bit_to_qam16`**: Convierte audio → símbolos QAM.  
- **`qam16_to_audio8bit`**: Reconstruye audio desde QAM.  

### **3.3 Evaluación**
- **Alineación**: Usar `alignsignals` antes de calcular SNR/SER.  
- **Reproducción**: `soundsc` para comparar audio original/recuperado.  

---

## **4. Preguntas Frecuentes**

1. **¿Por qué FIR de 9 coeficientes?**  
   - Suficiente para modelar memoria del canal sin excesiva complejidad.  

2. **¿Cómo afecta el ruido a la constelación?**  
   - Dispersa los puntos, aumentando errores.  

3. **¿Ventaja de NLMS sobre LS?**  
   - Adaptación en tiempo real a cambios del canal.  

---

## **5. Conclusión**  
Práctica que integra modulación QAM, ecualización (fija/adaptativa) y evaluación con señales reales de audio. Destaca la importancia de compensar ISI en canales multitrayectoria y la robustez de métodos adaptativos.  
