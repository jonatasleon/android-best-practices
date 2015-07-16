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

### Bibliotecas

**[Jackson](http://wiki.fasterxml.com/JacksonHome)** é uma biblioteca Java para converter Objetos em JSON e vice-versa. [Gson](https://code.google.com/p/google-gson/) é uma escolha comum para resolver esse problema, de qualquer forma nós achamos a Jackson mais performática, já que ela suporta outras alternativas para processar JSON: streaming, modelo de árvore em memória e a tradicional associação de dados JSON-POJO. Tendo em mente que, embora a Jackson seja uma biblioteca mais ampla do que a GSON, dependendo do seu caso, você pode preferir a GSON para evitar a limitação de 65 mil métodos. Outras alternativas: [Json-smart](https://code.google.com/p/json-smart/) e [Boon JSON](https://github.com/RichardHightower/boon/wiki/Boon-JSON-in-five-minutes)

**Network, caching e imagens.** Existem pares de soluções comprovadas para realização de requests para servidores que você deve considerar usar para implementação de seu próprio client. Use [Volley](https://android.googlesource.com/platform/frameworks/volley) ou [Retrofit](http://square.github.io/retrofit/). Volley também fornece helpers para carregar e fazer cache de imagens. Se você optar por usar a Retrofit, considere a biblioteca [Picasso](http://square.github.io/picasso/) para carregamento e cache de imagens, e para requisições HTTP eficientes veja a [OkHttp](http://square.github.io/okhttp/). Todas as três libs, Retrofit, Picasso e OkHttp, foram desenvolvidas pela mesma companhia, então elas se complementam muito bem entre sí. [OkHttp também pode ser usada junto a  Volley](http://stackoverflow.com/questions/24375043/how-to-implement-android-volley-with-okhttp-2-0/24951835#24951835).

**RxJava** é uma biblioteca para Programação Reativa, em outras palavras, manipulação de eventos assíncronos. Esse é um paradigma poderoso e promissor, que também pode ser confuso por ele ser muito diferente. Nós recomendamos que tome algum cuidado antes de usar essa biblioteca para arquitetar as entidades do aplicativo. Existem alguns projetos feitos por nós usando a RxJava, se você precisar de ajuda, fale com uma dessas pessoas: Timo Tuominen, Olli Salonen, Andre Medeiros, Mark Voit, Antti Lammi, Vera Izrailit, Juha Ristolainen. Nós escrevemos algumas postagens no blog sobre RxJava:  [[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android), [[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), [[4]](http://blog.futurice.com/android-development-has-its-own-swift).

Se você tiver nenhuma prévia experiência com Rx, comece aplicando-o apenas para responses da API. Como alternativa, comece usando-o para manipulação de simples eventos da UI, como eventos de clique ou digitação em um campo de busca. Se você está confiante em suas habilidades em Rx e quer aplica-las em uma arquitetura mais completa, então escreva Javadocs sobre todas as partes complicadas. Tenha em mente que outros programadores não familiarizados com RxJava podem levar um certo tempo fazendo a manutenção do projeto. Faça seu melhor para ajuda-los a entender seu código e também o Rx.

**[Retrolambda](https://github.com/evant/gradle-retrolambda)** é uma biblioteca Java para uso da sintaxe de expressões Lambda no Android e em outras plataformas anteriores ao JDK8. Ela ajuda a manter seu código compacto e legível, especialmente se você usa um estilo funcional como por exemplo RxJava. Para usa-la, instale o JDK8, defina-o como seu SDK local nas opções de _Project Structure_ do Android Studio, defina `JAVA8_HOME` e `JAVA7_HOME` como variavéis de ambiente, então no build.gradle na raiz do projeto defina:

```groovy
dependencies {
    classpath 'me.tatarka:gradle-retrolambda:2.4.1'
}
```

e no build.gradle de cada módulo adicione:

```groovy
apply plugin: 'retrolambda'

android {
    compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
}

retrolambda {
    jdk System.getenv("JAVA8_HOME")
    oldJdk System.getenv("JAVA7_HOME")
    javaVersion JavaVersion.VERSION_1_7
}
```

O Android Studio oferece suporte para lambdas do Java8. Se você é novo em lambdas, basta usar o _get started_ a seguir:

- Qualquer interface com apenas um método é _"lambda friendly"_ e pode ser envolvido dentro de uma sintaxe mais compacta
- Se você está em dúvida sobre os parâmetros e como escrever uma simples sub-classe anônima, então deixe que o Android Studio envolva-a em um lambda para você.

**Tenha cuidado com a limitação de métodos pelo dex e evite usar várias bibliotecas.** Aplicações Android quando empacotadas como um arquivo dex têm uma limitação de 65536 métodos a serem referenciados [[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/). Você irá ver um erro fatal durante a compilação se você ultrapassar esse limite. Por essa razão, use uma mínima quantidade de bibliotecas e a ferramenta [dex-method-counts](https://github.com/mihaip/dex-method-counts) para determinar quais conjuntos de bibliotecas pode ser utilizadas a fim de permanecer abaixo do limite. Evite especialmente usar a biblioteca Guava, já que ela contém mais de 13 mil métodos.
