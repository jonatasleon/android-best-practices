# Boas práticas em desenvolvimento Android

Lições aprendidas por desenvolvedores Android na [Futurice](http://www.futurice.com). Evite reinventar a roda
seguindo essas guidelines. Se você se interessa por desenvolvimento para iOS ou Windows Phone, não se esqueça
de checar também nosso documentos sobre [**iOS Good Practices**](https://github.com/futurice/ios-good-practices) e
 [**Windows App Development Best Practices**](https://github.com/futurice/windows-app-development-best-practices).

 [![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091)

## Sumário

#### Use Gradle e é recomendado estrutura projeto
#### Coloque senhas e dados sensíveis no gradle.properties
#### Não escreva seu próprio cliente HTTP, use as bibliotecas Volley ou OkHttp
#### Use a biblioteca Jackson para análise de dados JSON
#### Evite usar o Guava e use apenas algumas bibliotecas devido ao *limite de métodos de 65k*
#### Use Fragments para representar uma tela de UI
#### Use Activities só para gerenciar Fragments
#### Layouts XML são códigos, organize-os bem
#### Use styles para evitar atributos duplicados nos layouts XML
#### Use mútiplos arquivos de style para evitar um único grande arquivo
#### Mantenha seu colors.xml curto e seguindo DRY(Don't repeat yourself), apenas defina a paleta de cores
#### Também mantenha DRY para dimens.xml, defina constantes genéricas
#### Não faça uma longa hierarquia de ViewGroups
#### Evite processamento client-side para WebViews, e cuidado com vazamento de dados
#### Use Robolectric para unit tests, Robotium para connected (UI) tests
#### Use Genymotion como seu emulador


----------

### Android SDK

Coloque seu [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) em algum lugar no seu diretório home ou em algum outro lugar que seja independente de aplicações. Alguns IDEs incluem o SDK quando instalados e podem coloca-lo sob o mesmo diretório que eles. Isso pode ser ruim quando você precisar atualizar (ou reinstalar) o IDE ou quando estiver trocando de IDE. Também evite colocar o SDK em um diretório que a nível de sistema possa precisar de permissões sudo, se seu IDE está executando sob seu usuário e não sob o root.

### Build System

Sua opção padrão para Build System deve ser o [Gradle](http://tools.android.com/tech-docs/new-build-system). O Ant é muito mais limitado e também mais verboso. Com ele é simples:

- Contruir diferentes flavours ou variantes de seu aplicativo
- Fazer tarefas de script básico
- Gerenciar e baixar depedências
- Customizar keystores
- E mais

 O plugin do Gradle para Android está também sendo ativamente desenvolvido pela Google como o mais novo padrão para build system.

 ### Project structure
