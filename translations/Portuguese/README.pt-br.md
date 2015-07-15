# Boas práticas no desenvolvimento Android

Lições aprendidas por desenvolvedores Android na [Futurice](http://www.futurice.com). Evite reinventar a roda seguindo essas guidelines. Se você se interessa por desenvolvimento para iOS ou Windows Phone, também dê uma olhada em nossos documentos sobre [**iOS Good Practices**](https://github.com/futurice/ios-good-practices) e [**Windows App Development Best Practices**](https://github.com/futurice/windows-app-development-best-practices).

 [![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091)

## Sumário

#### Use o Gradle e é recomendado estrutura 'Project'
#### Mantenha senhas e dados sensíveis no gradle.properties
#### Não escreva seu próprio HTTP client, use as bibliotecas Volley ou OkHttp
#### Use a biblioteca Jackson para análise de dados JSON
#### Evite usar o Guava e utilize apenas algumas bibliotecas devido ao *limite de métodos em 65k*
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
#### Sempre use ProGuard ou DexGuard


----------

### Android SDK

Coloque seu [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) em algum lugar no seu diretório home ou em algum outro lugar que seja independente de aplicações. Alguns IDEs incluem o SDK quando instalados e podem coloca-lo sob o mesmo diretório que eles. Isso pode ser ruim quando você precisar atualizar (ou reinstalar) o IDE ou quando estiver trocando de IDE. Também evite colocar o SDK em um diretório que a nível de sistema possa precisar de permissões sudo, se seu IDE está executando sob seu usuário e não sob o root.

### Build System

Sua opção padrão para Build System deve ser o [Gradle](http://tools.android.com/tech-docs/new-build-system). O Ant é muito mais limitado e também mais verboso. Com ele é simples:

- Contruir diferentes distribuições ou variantes de seu aplicativo
- Fazer tarefas de script básico
- Gerenciar e baixar depedências
- Customizar keystores
- E mais

O plugin do Gradle para Android está também sendo ativamente desenvolvido pela Google como o mais novo padrão para build system.

### Estrutura de projeto

Existem dois tipos populares: a antiga estrutura de projeto Ant & Eclipse ADT e a nova estrutura Gradle & Android Studio. Você deve escolher a nova estrutura de projeto. Se seu projeto usa a antiga estrutura, considere-o legado e comece a porta-lo para a nova estrutura.

Antiga estrutura:

```
old-structure
├─ assets
├─ libs
├─ res
├─ src
│  └─ com/futurice/project
├─ AndroidManifest.xml
├─ build.gradle
├─ project.properties
└─ proguard-rules.pro
```

Nova estrutura:

```
new-structure
├─ library-foobar
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ com/futurice/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/futurice/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```

A principal diferença é que a nova estrutura separa explicitamente 'grupos de código fonte' (`main`, `androidTest`), um conceito do Gradle. Você poderia, por exemplo, adicionar grupos de código fonte 'pago' e 'free' dentro de `src` que terá o código fonte de seu aplicativo nas distruições paga e gratuita.

Tem um `app` em alto nível é útil para distinguir seu aplicativo de outros projetos de biblioteca (ex.: `library-foobar`) que serão referenciado no seu aplicativo. Então o `settings.gradle` mantém referências a estes projetos de biblioteca, que podem referenciar a `app/build.gradle`.

### Configuração do Gradle

**Estrutura Geral.** Siga [Guia da Google sobre Gradle para Android](http://tools.android.com/tech-docs/new-build-system/user-guide)

**Tarefas Pequenas.*** Ao invés de usar scripts (shell, Python, Perl etc), você pode realizar tarefas no Gradle. Para mais detalhes, siga a [documentação do Gradle](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF).

**Senhas.** No `build.gradle` do seu app você precisará definir as `signingConfigs` para lançar uma build. Aqui está o que você deve evitar:

_Não faça isso_. Dessa forma apareceria no sistema de controle de versão.

```groovy
signingConfigs {
    release {
        storeFile file("myapp.keystore")
        storePassword "password123"
        keyAlias "thekey"
        keyPassword "password789"
    }
}
```

Ao invés disso, crie um arquivo `gradle.properties` que _não_ deve ser adicionado ao sistema de controle de versão:

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```

Esse arquivo é automaticamente importado pelo gradle, então você pode usa-lo no `build.gradle`dessa forma:

 ```groovy
 signingConfigs {
     release {
         try {
             storeFile file("myapp.keystore")
             storePassword KEYSTORE_PASSWORD
             keyAlias "thekey"
             keyPassword KEY_PASSWORD
         }
         catch (ex) {
             throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
         }
     }
 }
 ```

**Prefira o Maven para solução de dependências ao invés de importar arquivos jar.** Se você explicitamente incluir arquivos jar no seu projeto, eles estarão congelados em alguma versão específica, tal como `2.1.1`. Fazer o download de jars e atualiza-los é um processo complicado, esse é um problema que o Maven se propôe a resolver e também é encorajado na criação do Android Gradle. Por exemplo:

```groovy
dependencies {
    compile 'com.squareup.okhttp:okhttp:2.2.0'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'
}
```

**Evite a solução de depencências dinâmicas do Maven**
Evite o uso de versionamento dinâmico, como `2.1.+`, pois isso pode resultar em builds instáveis ou sutilmente diferentes, diferenças não monitoradas no comportamento entre as builds. A utilização de versões estáticas, como `2.1.1` ajuda a criar um ambiente de desenvolvimento mais estável, previsível e repetitivo.

### IDEs e editores de texto

**Use qualquer editor de texto, mas ele deve ser agradável com a estrutura do projeto.** Editores de texto são uma escolha pessoal e é sua responsabilidade deixar seus editor funcionando de acordo com a estrutura do projeto e sistema de build.

O IDE mais recomendado atualmente é o Android Studio, por ser desenvolvido pela Google, está intímo com o Gradle, usa a nova estrutura de projeto por padrãoe, finalmente está em uma versão estável e é adaptador para desenvolvimento Android.

Você pode usar o [Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt) se desejar, mas você terá de configurá-lo, já que ele espera a antiga estrutura de projetos e o Ant para building. Você pode até usar um editor de texto plano, como o Vim, Sublime Text or Emacs. Nesse caso você deverá usar o Gradle e o `adb` por linha de comando. Se a integração do Eclipse com o Gradle não está funcionando para você, suas opções são usar a linha de comando somente para build ou migrar para o Android Studio. Essa é a melhor opção devido a recente depreciação do plugin ADT.

Qualquer um que você utilize basta ter certeza que o Gradle e a nova estrutura de projeto permanecem como a maneira oficial de construir a aplicação e evite adicionar ao sistema de controle de versão arquivos de configuração do seu editor em específico. Por exemplo, não adicione o arquivo do Ant, `build.xml`. Especialmente não se esqueça de manter o `build.gradle` atualizado e funcional se você estiver mudando as configurações de build no Ant. Também seja gentil com outros desenvolvedores, não force-os a mudar suas ferramentas de preferência.
