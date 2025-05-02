# Architettura e Implementazione Tecnica

## Indice
- [Panoramica](#panoramica)
- [Architettura Dettagliata](#architettura-dettagliata)
- [Moduli Principali](#moduli-principali)
- [Flusso dei Dati](#flusso-dei-dati)
- [Addestramento](#addestramento)
- [Inferenza](#inferenza)
- [Requisiti Computazionali](#requisiti-computazionali)
- [Limiti e Considerazioni](#limiti-e-considerazioni)
- [Riferimenti](#riferimenti)

## Panoramica

La rete neurale U-Net è qui utilizzata per il rilevamento di aree umide da immagini satellitari multispettrali. Il modello combina un'architettura encoder-decoder di tipo U-Net con un modulo di attenzione idrologica che incorpora indici specifici (NDWI, MNDWI) per migliorare la precisione del rilevamento.

## Architettura Dettagliata

L'architettura U-Net è composta dai seguenti componenti principali:

### 1. Encoder
- Backbone: ResNet34 pre-addestrato, modificato per accettare input a 6 canali (bande Sentinel-2)
- Strati di encoding progressivamente più profondi che estraggono caratteristiche a diverse scale spaziali (imagenet)

### 2. Decoder
- Blocchi di upsampling e convoluzione che ricostruiscono la risoluzione spaziale originale
- Skip connections che preservano informazioni spaziali dall'encoder

### 3. Modulo di Rilevamento Idrologico
- Calcolo di indici d'acqua (NDWI, MNDWI) direttamente dall'input multispettrale
- Elaborazione degli indici attraverso convoluzioni dedicate
- Fusione delle caratteristiche degli indici come mappe di warning
- Applicazione del warning alle caratteristiche del decoder

### 4. Strato di Output
- Convoluzione finale che produce una mappa di probabilità a singolo canale
- Attivazione sigmoide per normalizzare i valori di output tra 0 e 1

## Moduli Principali

### Compute Water Indices
```python
def compute_water_indices(self, x):
    # Calcolo di NDWI e MNDWI dalle bande multispettrali
```

Questa funzione calcola gli indici d'acqua NDWI e MNDWI direttamente dalle bande multispettrali dell'input. Questi indici sono utilizzati dal modulo di attenzione idrologica.

## Flusso dei Dati

1. **Input**: Immagine multispettrale con 6 bande (B, G, R, NIR, SWIR1, SWIR2)
   - Dimensioni: [B, 6, H, W]

2. **Preprocessing**:
   - Normalizzazione per canale utilizzando valori percentili (2% e 98%)
   - Calcolo degli indici d'acqua NDWI e MNDWI

3. **Encoder**:
   - Estrazione di caratteristiche attraverso il backbone U-Net
   - Generazione di feature maps a diverse risoluzioni

4. **Decoder**:
   - Upsampling progressivo con integrazioni di feature maps dall'encoder
   - Applicazione dell'attenzione idrologica

5. **Output**:
   - Mappa di probabilità a singolo canale delle aree umide
   - Dimensioni: [B, 1, H, W]

6. **Postprocessing**:
   - Sogliatura della mappa di probabilità per ottenere una maschera binaria

## Addestramento

I pesi pre-allenati sono scaricati da pytorch

## Inferenza

Per l'inferenza, il modello segue questi passaggi:

1. **Preprocessing**:
   - Normalizzazione delle bande di input
   - Ridimensionamento opzionale per gestire immagini di grandi dimensioni

2. **Forward Pass**:
   - Passaggio dell'immagine preprocessata attraverso la rete
   - Generazione della mappa di probabilità

3. **Postprocessing**:
   - Applicazione di una soglia (tipicamente 0.5) per ottenere una maschera binaria
   - Filtraggio opzionale per rimuovere piccoli oggetti o artefatti

## Requisiti Computazionali

- **Addestramento**:
in questa versione vengono scaricati i pesi da pytorch

- **Inferenza**:
  - GPU con 4GB+ di VRAM per immagini di dimensioni standard
  - CPU: possibile ma significativamente più lento
  - Tempo di inferenza: ~1-5 secondi per immagine di dimensione 1000x1000 su GPU

## Limiti e Considerazioni

### Limitazioni Tecniche

1. **Confusione con ombre**: Il modello può confondere le ombre scure con l'acqua, specialmente in aree montuose.
2. **Sensibilità alla copertura nuvolosa**: Nuvole e loro ombre possono ridurre l'accuratezza del rilevamento.
3. **Acque poco profonde**: Difficoltà nel rilevare acque molto poco profonde o con alta torbidità.

### Considerazioni per il Miglioramento

1. **Time-series data**: Incorporare dati multi-temporali per distinguere l'acqua permanente da quella temporanea.
2. **Modelli più grandi**: Utilizzare backbone più potenti (ResNet50, EfficientNet) per migliorare la capacità di estrazione di caratteristiche.
3. **Ensemble di modelli**: Combinare più modelli addestrati su diverse combinazioni di bande.

## Riferimenti

1. McFeeters, S.K. (1996). "The use of the Normalized Difference Water Index (NDWI) in the delineation of open water features". International Journal of Remote Sensing, 17(7), 1425–1432.
2. Xu, H. (2006). "Modification of normalised difference water index (NDWI) to enhance open water features in remotely sensed imagery". International Journal of Remote Sensing, 27(14), 3025–3033.
3. Ronneberger, O., Fischer, P., & Brox, T. (2015). "U-Net: Convolutional Networks for Biomedical Image Segmentation". Medical Image Computing and Computer-Assisted Intervention (MICCAI), 234–241.
4. He, K., Zhang, X., Ren, S., & Sun, J. (2016). "Deep Residual Learning for Image Recognition". IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 770–778.
5. Sentinel-2 User Handbook, European Space Agency, 2015.