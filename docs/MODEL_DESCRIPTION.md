# HydraNet: Architettura e Implementazione Tecnica

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

HydraNet è un'architettura di rete neurale progettata specificamente per il rilevamento di aree umide da immagini satellitari multispettrali. Il modello combina un'architettura encoder-decoder di tipo U-Net con un modulo di attenzione idrologica che incorpora indici d'acqua specializzati (NDWI, MNDWI) per migliorare la precisione del rilevamento.

La caratteristica distintiva di HydraNet è l'integrazione diretta della conoscenza idrologica nella struttura della rete attraverso un meccanismo di attenzione che si concentra sulle caratteristiche spettrali tipiche dell'acqua.

## Architettura Dettagliata

L'architettura HydraNet è composta dai seguenti componenti principali:

### 1. Encoder
- Backbone: ResNet18 pre-addestrato, modificato per accettare input a 6 canali (bande Sentinel-2)
- Strati di encoding progressivamente più profondi che estraggono caratteristiche a diverse scale spaziali

### 2. Decoder
- Blocchi di upsampling e convoluzione che ricostruiscono la risoluzione spaziale originale
- Skip connections che preservano informazioni spaziali dall'encoder

### 3. Modulo di Attenzione Idrologica
- Calcolo di indici d'acqua (NDWI, MNDWI) direttamente dall'input multispettrale
- Elaborazione degli indici attraverso convoluzioni dedicate
- Fusione delle caratteristiche degli indici come mappe di attenzione
- Applicazione dell'attenzione alle caratteristiche del decoder

### 4. Strato di Output
- Convoluzione finale che produce una mappa di probabilità a singolo canale
- Attivazione sigmoid per normalizzare i valori di output tra 0 e 1

## Moduli Principali

### HydraNetConvBlock
```python
class HydraNetConvBlock(nn.Module):
    def __init__(self, in_channels, out_channels):
        # Implementazione di un blocco convolutivo standard
        # Due convoluzioni 3x3 con BatchNorm e ReLU
```

Questo modulo implementa un blocco convolutivo standard utilizzato sia nell'encoder che nel decoder, composto da due convoluzioni 3x3 seguite da normalizzazione batch e attivazione ReLU.

### HydraNetUpBlock
```python
class HydraNetUpBlock(nn.Module):
    def __init__(self, in_channels, out_channels):
        # Implementazione di un blocco di upsampling
        # Deconvoluzione + concatenazione con skip connection + blocco convolutivo
```

Questo modulo implementa un blocco di upsampling utilizzato nel decoder. Esegue una deconvoluzione per aumentare la risoluzione spaziale, concatena il risultato con una feature map dall'encoder (skip connection) e applica un blocco convolutivo.

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
   - Estrazione di caratteristiche attraverso il backbone ResNet
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

### Dataset

Per addestrare efficacemente HydraNet, è necessario un dataset con le seguenti caratteristiche:

- **Input**: Immagini Sentinel-2 con 6 bande (B, G, R, NIR, SWIR1, SWIR2)
- **Target**: Maschere binarie che indicano la presenza di acqua
- **Dimensioni consigliate**: Almeno 1000 coppie immagine-maschera per risultati robusti
- **Diversità**: Varietà di tipi di acqua (laghi, fiumi, bacini artificiali) e condizioni ambientali

### Funzione di Loss

La funzione di loss raccomandata è una combinazione di:

- **Binary Cross-Entropy (BCE)**: Per la classificazione pixel-wise
- **Dice Loss**: Per gestire lo sbilanciamento delle classi (l'acqua spesso occupa una piccola porzione dell'immagine)

```python
def combined_loss(y_pred, y_true):
    bce = F.binary_cross_entropy(y_pred, y_true)
    dice = 1 - (2 * (y_pred * y_true).sum() + 1e-5) / (y_pred.sum() + y_true.sum() + 1e-5)
    return 0.5 * bce + 0.5 * dice
```

### Parametri di Training

- **Ottimizzatore**: Adam con learning rate iniziale di 1e-4
- **Scheduler**: ReduceLROnPlateau per ridurre il learning rate quando la loss si stabilizza
- **Batch size**: 8-16, a seconda della memoria GPU disponibile
- **Epoche**: 50-100, con early stopping basato sulla validation loss
- **Data augmentation**: Rotazioni, flips, variazioni di luminosità e contrasto

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
  - GPU con almeno 8GB di VRAM
  - 16GB+ di RAM del sistema
  - Tempo di addestramento stimato: 8-24 ore su una singola GPU moderna

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