# Spalla Flutter SDK

Player Spalla para aplicativos Flutter Android e iOS, com controles
compartilhados, fullscreen, selecao de faixas, anuncios, Cast e PiP.

Versao: **0.1.0**.

## Disponibilidade

O pacote esta disponivel no [pub.dev](https://pub.dev/packages/spalla_flutter).
Use as instrucoes de instalacao abaixo para adicionar o SDK ao aplicativo.
Este repositorio contem somente documentacao para integradores, licenca
e suporte via issues. Nao e um checkout do pacote Flutter.

## Requisitos

| Plataforma | Requisito |
| --- | --- |
| Flutter | >= 3.27 |
| Dart | >= 3.6 |
| Android | API >= 23; PiP requer API >= 26 |
| iOS | >= 15.0, CocoaPods |

Obtenha seu token SDK e os identificadores de conteudo na Spalla.
Para Cast, use o application ID do receiver configurado para sua integracao.

## Instalacao

```yaml
dependencies:
  spalla_flutter: ^0.1.0
```

Execute `flutter pub get` no aplicativo. Depois de adicionar o plugin,
recompile o app; hot reload nao instala plugins nativos.

## Android

Para Cast, use FlutterFragmentActivity na Activity do aplicativo:

```kotlin
import io.flutter.embedding.android.FlutterFragmentActivity

class MainActivity : FlutterFragmentActivity()
```

Para PiP, configure a Activity no AndroidManifest.xml:

```xml
<activity
    android:name=".MainActivity"
    android:supportsPictureInPicture="true"
    android:resizeableActivity="true"
    android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode" />
```

O SDK inclui as permissoes INTERNET/ACCESS_NETWORK_STATE e um OptionsProvider
de Cast. Se o aplicativo ja tem um OptionsProvider, concilie a configuracao
para nao declarar dois providers. Informe o application ID antes de usar Cast.

## iOS

Use iOS 15.0 ou superior no projeto/Podfile. Para Cast, adicione ao Info.plist:

```xml
<key>NSLocalNetworkUsageDescription</key>
<string>Descobrir dispositivos Cast na rede local.</string>
<key>NSBonjourServices</key>
<array>
    <string>_googlecast._tcp</string>
    <string>_SEU_CAST_APP_ID._googlecast._tcp</string>
</array>
```

Para PiP/reproducao em background, habilite Background Modes > Audio,
AirPlay and Picture in Picture e inclua:

```xml
<key>UIBackgroundModes</key>
<array><string>audio</string></array>
```

Nao desabilite ATS globalmente. Se um provedor exigir HTTP, configure apenas
as excecoes necessarias para os dominios utilizados pelo aplicativo.

## Uso Basico

```dart
import 'package:flutter/material.dart';
import 'package:spalla_flutter/spalla_flutter.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  initialize('SEU_TOKEN_SDK', 'SEU_CAST_APP_ID');
  runApp(const MaterialApp(home: PlayerScreen()));
}

class PlayerScreen extends StatelessWidget {
  const PlayerScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    body: Center(
      child: AspectRatio(
        aspectRatio: 16 / 9,
        child: SpallaPlayer(
          contentId: 'SEU_CONTENT_ID',
          autoplay: true,
          pipEnabled: true,
          onPlayerEvent: (event) => debugPrint(event.event),
        ),
      ),
    ),
  );
}
```

O application ID e opcional para reproducao local. Inicialize o SDK antes
de montar o primeiro player. Use um token SDK apropriado para o cliente,
nunca credenciais administrativas. Nao registre tokens ou URLs autenticadas.

## Opcoes Do Player

| Opcao | Uso |
| --- | --- |
| `contentId` | Identificador Spalla do conteudo |
| `controller` | Controle externo opcional |
| `autoplay` | Reproducao automatica, default true |
| `muted` | Comecar sem som, default false |
| `startTime` | Posicao inicial em segundos |
| `subtitle` | Idioma/nome da legenda ou null |
| `audioTrack` | Idioma/nome da faixa de audio |
| `playbackRate` | 0.25, 0.5, 1, 1.25, 1.5 ou 2 |
| `hideUI` | Ocultar controles padrao |
| `autoCdnSwitch` | Recuperacao automatica de CDN, default true |
| `pipEnabled` | Habilitar PiP, default false |
| `playInBackground` | Manter reproducao em background, default false |
| `aspectRatio` | fit, fill ou aspectFill |
| `presentationMode` | inline ou pip |
| `subtitleAppearance` | Tamanho/margem das captions |
| `customAds` / `customImaParams` | Anuncios e parametros IMA customizados |
| `children` | Widgets adicionais sobre o player |
| `onPlayerEvent` | Callback de eventos |

## Controlador

Passe a mesma instancia de SpallaPlayerController em `controller` do widget.
Comandos de reproducao exigem `controller.isAttached == true`. Uma instancia
pertence a um unico player. Descarte o controlador externo depois de remover
o widget; o controlador interno e gerenciado automaticamente.

| Metodo | Retorno |
| --- | --- |
| `load(id, isLiveContent, autoPlay, startTime, subtitle)` | Future<void> |
| `play()`, `pause()`, `stop()` | Future<void> |
| `seekTo(seconds)`, `seekToLive()` | Future<void> |
| `getDuration()`, `getCurrentTime()` | double, segundos |
| `setMuted(bool)` | Future<void> |
| `selectSubtitle(String?)` | Future<void> |
| `setAudioTrack(String?)` | Future<void> |
| `selectPlaybackRate(double)` | Future<void> |
| `setBitrate(int?)` | Future<void>; null seleciona Auto |
| `enterFullScreen()`, `exitFullScreen()` | Future<void> |
| `enterPictureInPictureMode()` | Future<bool> |
| `onPictureInPictureModeChanged(bool)` | Future<void> |
| `showCastDialog()` | Future<void> |
| `onPause()`, `onResume()`, `onDestroy()` | Future<void> |
| `isDestroyed()` | bool |
| `registerPlayerListener(listener)` | void |
| `registerFullScreenListener(listener)` | void |
| `registerCastListener(listener)` | void |

`setSubtitle`/`setPlaybackRate` sao aliases de selecao. `enterPiP` tambem
esta disponivel. Listeners registrados coexistem com `onPlayerEvent` e
`controller.events`.

`load` troca o conteudo do player montado. O parametro isLiveContent e aceito
por compatibilidade; a configuracao do conteudo determina live/VOD.
`stop` encerra a reproducao; use `load` para preparar novamente.
`onResume` retoma apenas se estava tocando antes de `onPause`. `onDestroy`
e terminal e idempotente. Getters retornam o ultimo estado recebido.

## Eventos

Eventos possuem `event` e payload em `data`/`toMap()`. `nativeEvent` tambem
permite acesso ao envelope. Principais nomes:

- play, pause, playing, buffering, ended, muted, unmuted.
- metadataLoaded, durationUpdate, timeUpdate.
- subtitlesAvailable, subtitleSelected, audioTracksAvailable, audioTrackSelected.
- playbackRateSelected, thumbnailsAvailable, cdnChanged.
- onEnterFullScreen, onExitFullScreen, enterPiP, exitPiP, castStateChanged.
- adBreakBegin, adBegin, adEnd, adBreakEnd, adEvent, adError.
- integrationWarning.

TimeUpdate inclui tempo e, quando disponiveis, janela seekable, latencia live
e atLiveEdge. Eventos de anuncio incluem ciclo semantico e dados brutos IMA.
Para diagnosticar a instalacao, use `await checkIntegration()`.

## Controles E Legendas

A mesma skin e utilizada no Android/iOS, inclusive em fullscreen. Toque
revela controles; double-tap busca 10 segundos. Opcoes incluem velocidade,
legendas, audio, qualidade e PiP. Fullscreen respeita a orientacao do app.

```dart
subtitleAppearance: const SubtitleAppearance(
  fontSize: 16,
  bottomPaddingRatio: 1.8,
  fullscreen: SubtitleSizeConfig(fontSize: 24),
),
```

Legendas HLS embutidas tem prioridade sobre SRT do mesmo idioma. SRT em
overlay acompanha fullscreen, mas nao aparece em PiP nativo.
Para composicao personalizada, use hideUI e SpallaPlayerControls com o mesmo
controlador. SpallaCastButton recebe controller e tintColor opcional.

## Anuncios

```dart
customAds: const [
  CustomAd(url: 'https://ads.example.com/preroll', offset: 'start'),
  CustomAd(url: 'https://ads.example.com/midroll', offset: 60),
],
customImaParams: const {'cust_params': 'category=sports'},
```

Offsets aceitam start/pre, end/post, segundos, percentual ou hh:mm:ss.mmm.
CustomAds substituem VAST/VMAP; DAI continua vindo da configuracao Spalla.
Nao coloque widgets que bloqueiem os toques da interface de anuncios.

## Limitacoes Da Versao

- Android/iOS apenas. PiP depende do suporte e da configuracao do sistema.
- Overlay SRT nao esta disponivel em PiP; legendas nativas continuam no player.
- Thumbnails sao expostos por evento; nao ha preview de sprite na barra atual.
- Cast com receiver fisico, anuncios/DAI com fill, live/DVR e background
  prolongado devem ser validados no aplicativo e nos dispositivos alvo.
- A versao 0.1.0 nao representa certificacao de equivalencia integral com
  outros SDKs Spalla.

## Suporte E Licenca

Abra uma issue com versao do SDK/Flutter, plataforma, versao do sistema,
passos de reproducao e eventos/erros sanitizados. Nao inclua tokens,
credenciais administrativas nem URLs contendo autenticacao.

https://github.com/taghos/spalla-flutter-sdk-releases/issues

O SDK e distribuido sob os termos Taghos em [LICENSE](LICENSE).
Avisos de terceiros: [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).