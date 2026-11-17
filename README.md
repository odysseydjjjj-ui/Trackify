# Trackify

Open-source музыкальный плеер на Flutter с нативным интерфейсом (без HTML/WebView). Текущий этап: desktop UI + подключения SoundCloud и Discord Rich Presence.

## Что работает в этом этапе

- SoundCloud OAuth 2.1 Authorization Code + PKCE.
- Получение профиля авторизованного SoundCloud-пользователя через `GET /me`.
- Хранение токенов и credentials через `flutter_secure_storage`.
- Discord Rich Presence через Discord Desktop RPC.
- Экран Connections в боковом меню.

> На данном этапе подключение SoundCloud подтверждает аккаунт и загружает его профиль. Поиск и потоковое воспроизведение SoundCloud, синхронизация настоящих метаданных трека с Discord, автообновление, трей и мобильный клиент пока не реализованы.

## SoundCloud setup

1. Создай API application в [SoundCloud Developers](https://soundcloud.com/you/apps/).
2. Скопируй `Client ID` и `Client Secret` в Trackify → Connections → SoundCloud.
3. Добавь redirect URI `http://127.0.0.1:43821/callback` в настройки приложения SoundCloud.
4. Нажми **CONNECT SOUNDCLOUD** и подтверди доступ в открывшемся браузере.

SoundCloud использует OAuth 2.1 + PKCE. API требует `client_secret`; он сейчас вводится пользователем и хранится в системном защищённом хранилище. **Перед публичным распространением EXE токен-обмен следует вынести в собственный backend**, чтобы не раздавать секрет приложения в клиентском бинарнике. Соблюдай SoundCloud API Terms и attribution requirements.

## Discord setup

1. Создай приложение в [Discord Developer Portal](https://discord.com/developers/applications).
2. Скопируй Application ID из General Information.
3. Убедись, что Discord Desktop запущен.
4. В Trackify → Connections → Discord Rich Presence введи Application ID и нажми **ENABLE ACTIVITY**.

Rich Presence работает только с Discord Desktop на Windows в текущей конфигурации. Загрузи asset с ключом `trackify` в Discord Developer Portal → Rich Presence → Art Assets, чтобы отображалась своя иконка. Для Discord RPC мобильного клиента нет.

## Build on Windows

Требуется Flutter stable и Visual Studio с workload **Desktop development with C++**.

```powershell
flutter create --platforms=windows,android .
flutter pub get
flutter run -d windows
flutter build windows --release
```

EXE и файлы runtime будут в `build/windows/x64/runner/Release/`.

## Notes

- Не коммить API credentials, токены и файлы `.env`.
- `flutter_secure_storage` использует системное хранилище секретов.
- SoundCloud API может ограничивать доступность отдельных треков и функций; соблюдай API Terms и требования к атрибуции.
