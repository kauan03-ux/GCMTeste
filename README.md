# GameSir Test App — Super Nova / T3XS

Aplicativo Android de diagnóstico e testes BLE para o controle **GameSir Super Nova / T3XS**.

Versão 1.0 — baseado no guia completo de 14/10/2026.

## O que o app faz

- Escaneia e conecta ao controle via **Bluetooth Low Energy**
- Enumera todos os serviços, characteristics e propriedades GATT
- Console GCM com presets conhecidos e entrada hexadecimal
- Logger TX/RX com timestamps
- **Modo somente leitura por padrão** (SET e RESET bloqueados)
- Desbloqueio experimental com confirmação
- Confirmação dupla antes de qualquer Reset
- Exportação de logs (copia para área de transferência)
- Testes unitários do codec GCM (checksum dos exemplos do guia)

## Estrutura do projeto

```
app/src/main/java/com/gamesir/testapp/
├── MainActivity.kt
├── model/Models.kt          # Pacotes GCM, logs, estados
├── protocol/gcm/GcmCodec.kt # Codec + presets + validação
├── ble/BleManager.kt        # Scan, GATT, escrita, notificações
└── ui/
    ├── MainViewModel.kt
    └── Screens.kt           # Dashboard, GATT, Console, Logger
```

## Como compilar (sem Android Studio instalado antes)

### Opção recomendada — Android Studio (gratuito)

1. Baixe e instale o **Android Studio**:  
   https://developer.android.com/studio

2. Abra o Android Studio → **Open** → selecione a pasta `GameSirTestApp`

3. Na primeira abertura ele vai baixar o SDK e as dependências (pode demorar alguns minutos)

4. Conecte um celular com USB Debugging ativado **ou** use um emulador

5. Clique em **Run** (▶) ou gere o APK:
   - Menu **Build → Build Bundle(s) / APK(s) → Build APK(s)**
   - O APK fica em `app/build/outputs/apk/debug/app-debug.apk`

6. Para assinar um release (você já disse que consegue):
   - **Build → Generate Signed Bundle / APK**

### Opção linha de comando (se já tiver JDK 17+ e Android SDK)

```bash
cd GameSirTestApp
# Se não tiver o wrapper:
# gradle wrapper --gradle-version 8.9

./gradlew assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```

## Permissões

O app pede:

- `BLUETOOTH_SCAN` + `BLUETOOTH_CONNECT` (Android 12+)
- Localização apenas em Android ≤ 11 (exigência antiga da API de scan BLE)

## Uso seguro (resumo do guia)

1. Comece sempre em **somente leitura**
2. Escaneie → conecte → veja o inventário GATT
3. Rode **Query** e observe as respostas no Logger
4. Só desbloqueie o modo experimental depois de entender o que está fazendo
5. Nunca comece por Reset
6. Antes de um Set, anote o estado anterior
7. Depois de um Set, faça read-back e, se possível, reconecte e consulte de novo

## Protocolo GCM (formato observado)

```
1F | OP | ID | VALUE | 00 | CHECKSUM
```

| OP   | Significado |
|------|-------------|
| 0x00 | Query       |
| 0x01 | Set         |
| 0x02 | Reset       |

Checksum = soma dos bytes anteriores, módulo 256.

Exemplos validados nos testes unitários:

| Bytes                  | Interpretação          |
|------------------------|------------------------|
| `1F 00 01 00 00 20`    | Query ID 01            |
| `1F 01 01 C1 00 E2`    | Set ID 01 = C1         |
| `1F 02 01 00 00 22`    | Reset ID 01            |
| `1F 01 01 B7 00 D8`    | Set ID 01 = B7         |

## UUIDs conhecidos (pistas do SDK)

- Serviço: `00008650-0000-1000-8000-00805f9b34fb`
- Characteristic (Handle Data): `00008655-0000-1000-8000-00805f9b34fb`
- Outra: `00008651-0000-1000-8000-00805f9b34fb`

O app enumera o GATT real do dispositivo e tenta selecionar a characteristic conhecida. Se o Super Nova usar outra, você verá no explorador GATT e pode usar a entrada hexadecimal livre.

## Limitações atuais (conforme o guia)

- Characteristic GCM exata do T3XS ainda **pendente de confirmação física**
- IDs de todos os botões/teclas são parciais
- Persistência dos mapeamentos **não validada** fisicamente
- Frames PC (A5/C0) **não são usados** como pacotes BLE

## Testes unitários

```bash
./gradlew test
```

Cobre checksum dos exemplos, pacotes truncados, checksum inválido e round-trip de parse.

## Licença / aviso

Este é um **test harness de investigação**. Não é firmware oficial nem ferramenta de suporte GameSir. Use por sua conta e risco. Não faça fuzzing nem atualização de firmware com este app.
