# Running Maestro UI tests for iOS on Bitrise CI/CD

## bitrise.yml

```yaml
---
format_version: '11'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: ios
workflows:
  primary:
    description: |
      The workflow executes the tests. The *retry_on_failure* test repetition mode is enabled.

      Next steps:
      - Check out [Getting started with iOS apps](https://devcenter.bitrise.io/en/getting-started/getting-started-with-ios-apps.html).
    steps:
    - activate-ssh-key@4: {}
    - git-clone@8: {}
    - xcode-build-for-simulator@0:
        inputs:
        - simulator_device: iPhone 14
    - script@1:
        inputs:
        - content: |-
            #!/bin/sh

            set -ex

            # Maestro version
            if [[ -z "$maestro_cli_version" ]]; then
                echo "If you want to install a specific version of Maestro CLI, please set the environment variable maestro_cli_version to the version you want to install."
            else
                echo "Installing Maestro CLI version $maestro_cli_version"
                export MAESTRO_VERSION=$maestro_cli_version;
            fi

            # Install maestro CLI
            echo "Installing Maestro CLI"
            curl -Ls "https://get.maestro.mobile.dev" | bash
            export PATH="$PATH":"$HOME/.maestro/bin"
            echo "MAESTRO INSTALLED - Check Version"
            maestro -v


            echo "Installing Maestro dependencies"
            brew tap facebook/fb
            brew install facebook/fb/idb-companion

            xcrun simctl boot "iPhone 14"
            xcrun simctl install booted "${BITRISE_APP_DIR_PATH}"

            cd $PROJECT_LOCATION

            ls

            pwd


            echo "Running tests with Maestro"
            maestro test .maestro/flow.yaml --format junit --output "${BITRISE_DEPLOY_DIR}/flow-report.xml"
    - deploy-to-bitrise-io@2: {}
meta:
  bitrise.io:
    stack: osx-xcode-14.3.x-ventura
    machine_type_id: g2-m1.4core
app:
  envs:
  - opts:
      is_expand: false
    BITRISE_PROJECT_PATH: maestro-bitrise-ios.xcodeproj
  - opts:
      is_expand: false
    BITRISE_SCHEME: maestro-bitrise-ios
  - opts:
      is_expand: false
    BITRISE_DISTRIBUTION_METHOD: development
  - opts:
      is_expand: false
    PROJECT_LOCATION: "."
trigger_map:
- push_branch: main
  workflow: primary
```