# mirroring-template

Набор GitLab CI/CD шаблонов для зеркалирования артефактов между разными источниками.

Основная задача репозитория — предоставить переиспользуемые pipeline templates, которые можно подключать в другие проекты через `include:` и выполнять синхронизацию.

---

## oci-repository

### inputs

| Параметр         | Описание                               | Тип       | По умолчанию                           | Обязательный |
|------------------|----------------------------------------|-----------|----------------------------------------|--------------|
| `oci-repository` | Enable oci repository                  | `boolean` | `true`                                 | Нет          |
| `job-prefix`     | Job prefix                             | `string`  | `-$CI_PROJECT_ID-$CI_COMMIT_SHORT_SHA` | Нет          |
| `skopeo-image`   | cosign version to download             | `string`  | `quay.io/containers/skopeo:latest`     | Нет          |
| `stage`          | Job stage                              | `string`  | `publish`                              | Нет          |
| `src-auth-file`  | Source auth file in docker format      | `string`  | `~/.docker/config.json`                | Нет          |
| `dest-auth-file` | Destination auth file in docker format | `string`  | `~/.docker/config.json`                | Нет          |
| `skopeo-args`    | Extra args for skopeo                  | `string`  | `""`                                   | Нет          |
| `src-oci-image`  | Source image (support env)             | `string`  | —                                      | Да           |
| `dest-oci-image` | Destination image (support env)        | `string`  | —                                      | Да           |

### ENV

| Переменная окружения | Значение                       | Назначение                                                                                                                 |
|----------------------|--------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| `SKOPEO_IMAGE`       | `$[[ inputs.skopeo-image ]]`   | Определяет контейнерный образ с утилитой `skopeo`.                                                                         |
| `SRC_AUTH_FILE`      | `$[[ inputs.src-auth-file ]]`  | Путь к Docker auth-файлу для аутентификации в исходном OCI/Docker registry.                                                |
| `DEST_AUTH_FILE`     | `$[[ inputs.dest-auth-file ]]` | Путь к Docker auth-файлу для аутентификации в целевом OCI/Docker registry.                                                 |
| `SKOPEO_OCI_ARGS`    | `$[[ inputs.skopeo-args ]]`    | Дополнительные аргументы командной строки для `skopeo` (например TLS-настройки, debug-режим, параметры транспорта и т.д.). |
| `SRC_OCI_IMAGE`      | `$[[ inputs.src-oci-image ]]`  | Исходный OCI-образ/репозиторий.                                                                                            |
| `DEST_OCI_IMAGE`     | `$[[ inputs.dest-oci-image ]]` | Целевой OCI-образ/репозиторий.                                                                                             |


### Blocs

| Расширение / Блок                                | Назначение                                                                                     |
|--------------------------------------------------|------------------------------------------------------------------------------------------------|
| `.mirroring:oci-repository:public:env-overwrite` | Переопределение переменных окружения `SRC_OCI_IMAGE` и `DEST_OCI_IMAGE` при сложной генерации. |
| `.mirroring:oci-repository:public:rules-extends` | Переопределение правил запуска.                                                                |
| `.mirroring:oci-repository:public:extends`       | Переопределение блоков mirroring job.                                                          |
| `.mirroring:oci-repository:public:needs`         | Определяет зависимости mirroring от других job.                                                |

### Использование

```yaml
include:
  - component: $CI_SERVER_FQDN/library/cicd/templates/mirroring/oci-repository@main
    inputs:
      stage: "test"
      src-auth-file: "/tmp/config.json"
      dest-auth-file: "/tmp/config.json"
      src-oci-image: "$EXAMPLE_ENV_SRC"
      dest-oci-image: "$EXAMPLE_ENV_DEST"

variables:
  EXAMPLE_ENV_SRC: "stc/image"
  EXAMPLE_ENV_DEST: "dect/image"

stages:
  - test

.mirroring:oci-repository:public:env-overwrite:
  - SRC_OCI_IMAGE="$[[ inputs.src-oci-image ]]"
  - DEST_OCI_IMAGE="$[[ inputs.dest-oci-image ]]"

.mirroring:oci-repository:public:rules-extends:
  tags:
    - docker-builder

```
