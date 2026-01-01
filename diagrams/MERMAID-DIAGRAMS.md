# Pelican Cinema - System Diagrams (Mermaid)

## System Overview

```mermaid
flowchart LR
    subgraph Sources["📺 Sources"]
        PS5[PS5]
        Xbox[Xbox]
        ATV[Apple TV]
        Bluray[Blu-ray]
    end
    
    subgraph Command["🎛️ Command Module"]
        Switch[HDMI Switch<br/>4×1]
        Extract[Audio<br/>Extractor]
        Pi[Raspberry Pi 5]
        TX[ESP32<br/>Master TX]
    end
    
    subgraph Wireless["📡 Wireless"]
        ESPNOW((ESP-NOW<br/>2.4GHz<br/>Mesh))
    end
    
    subgraph Speakers["🔊 Speaker Pods"]
        FL[FL]
        FR[FR]
        C[Center]
        SUB[Subwoofer]
        SL[SL]
        SR[SR]
        RL[RL]
        RR[RR]
    end
    
    PS5 --> Switch
    Xbox --> Switch
    ATV --> Switch
    Bluray --> Switch
    
    Switch --> Extract
    Extract --> Pi
    Pi --> TX
    TX --> ESPNOW
    
    ESPNOW --> FL
    ESPNOW --> FR
    ESPNOW --> C
    ESPNOW --> SUB
    ESPNOW --> SL
    ESPNOW --> SR
    ESPNOW --> RL
    ESPNOW --> RR
```

## Signal Flow

```mermaid
flowchart TD
    A[🎮 HDMI Sources] --> B[HDMI Switch]
    B --> C[Audio Extractor]
    C --> D[7.1 PCM Audio]
    D --> E[Raspberry Pi 5]
    E --> F[Channel Separation]
    F --> G[ESP32 Master TX]
    G --> H{ESP-NOW Broadcast}
    
    H --> I1[Pod FL]
    H --> I2[Pod FR]
    H --> I3[Pod C]
    H --> I4[Pod SL]
    H --> I5[Pod SR]
    H --> I6[Pod RL]
    H --> I7[Pod RR]
    H --> I8[Pod SUB]
    
    subgraph Each Pod
        J[ESP32 RX] --> K[PCM5102A DAC]
        K --> L[TPA3116 Amp]
        L --> M[Speaker Driver]
    end
```

## Speaker Pod Internals

```mermaid
flowchart LR
    subgraph Wireless
        RX[ESP32-S3<br/>Receiver]
    end
    
    subgraph Audio
        DAC[PCM5102A<br/>DAC]
        AMP[TPA3116<br/>Class-D Amp]
        SPK[4" Full Range<br/>Driver]
    end
    
    subgraph Power
        BAT[3S LiFePO4<br/>9.6V 6Ah]
        BMS[Battery<br/>BMS]
        BUCK[5V Buck<br/>Converter]
    end
    
    RX -->|I2S| DAC
    DAC -->|Analog| AMP
    AMP -->|Power| SPK
    
    BAT --> BMS
    BMS -->|9.6V| AMP
    BMS --> BUCK
    BUCK -->|5V| RX
    BUCK -->|5V| DAC
```

## 7.1 Speaker Layout

```mermaid
flowchart TB
    subgraph Screen["🎬 Screen"]
        direction LR
        SCRN[" "]
    end
    
    subgraph Front["Front Speakers"]
        direction LR
        FL((FL))
        C((C))
        FR((FR))
    end
    
    subgraph Surround["Surround Speakers"]
        direction LR
        SL((SL))
        SR((SR))
    end
    
    subgraph Rear["Rear Speakers"]
        direction LR
        RL((RL))
        RR((RR))
    end
    
    subgraph Sub["Subwoofer"]
        SUB((SUB))
    end
    
    subgraph Listener["👤 Listening Position"]
        LP[" "]
    end
    
    SCRN ~~~ Front
    Front ~~~ Listener
    SL ~~~ LP
    SR ~~~ LP
    LP ~~~ Rear
    SUB ~~~ Front
```

## Phase Workflow

```mermaid
flowchart LR
    P1[Phase 1<br/>Wireless PoC<br/>$45] --> G1{Go?}
    G1 -->|Yes| P2[Phase 2<br/>Multi-Ch Sync<br/>$55]
    P2 --> G2{Go?}
    G2 -->|Yes| P3[Phase 3<br/>Command Module<br/>$620]
    P3 --> G3{Go?}
    G3 -->|Yes| P4[Phase 4<br/>Prototype Pod<br/>$175]
    P4 --> G4{Go?}
    G4 -->|Yes| P5[Phase 5<br/>Production<br/>$1,360]
    P5 --> P6[Phase 6<br/>Case Integration<br/>$350]
    P6 --> P7[Phase 7<br/>Calibration<br/>$50]
    P7 --> DONE[🎉 Complete!]
    
    G1 -->|No| PIVOT1[Pivot/Redesign]
    G2 -->|No| PIVOT2[Pivot/Redesign]
    G3 -->|No| PIVOT3[Pivot/Redesign]
    G4 -->|No| PIVOT4[Iterate Design]
```

## Build Dependencies

```mermaid
flowchart TD
    D11[D-1.1<br/>ESP32 Setup] --> D12[D-1.2<br/>I2S Audio]
    D12 --> D13[D-1.3<br/>ESP-NOW Stream]
    D13 --> CHECK1{Phase 1<br/>Checkpoint}
    
    CHECK1 --> D21[D-2.1<br/>Second RX]
    D21 --> D22[D-2.2<br/>Sync Protocol]
    D22 --> D23[D-2.3<br/>Measure Sync]
    D23 --> CHECK2{Phase 2<br/>Checkpoint}
    
    CHECK2 --> D31[D-3.1<br/>HDMI Setup]
    D31 --> D32[D-3.2<br/>Pi 5 Platform]
    D32 --> D33[D-3.3<br/>Channel Routing]
    D33 --> D34[D-3.4<br/>Master TX]
    D34 --> D35[D-3.5<br/>Enclosure]
    D35 --> CHECK3{Phase 3<br/>Checkpoint}
```

## Data Flow Timing

```mermaid
sequenceDiagram
    participant S as Source
    participant P as Pi 5
    participant TX as ESP32 TX
    participant RX as ESP32 RX
    participant D as DAC
    participant SP as Speaker
    
    S->>P: HDMI Audio
    Note over S,P: ~5ms
    P->>TX: I2S/SPI
    Note over P,TX: ~2ms
    TX->>RX: ESP-NOW
    Note over TX,RX: ~3ms
    RX->>RX: Jitter Buffer
    Note over RX: ~8ms
    RX->>D: I2S
    Note over RX,D: ~2ms
    D->>SP: Analog
    Note over D,SP: ~1ms
    Note over S,SP: Total: ~21ms
```
