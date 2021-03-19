## JavaFX

# 1. Introdução
JavaFX é uma biblioteca para construir aplicativos de cliente ricos com Java. Ele fornece uma API para projetar aplicativos GUI que são executados em quase todos os dispositivos com suporte a Java.

Neste tutorial, vamos nos concentrar e cobrir alguns de seus principais recursos e funcionalidades.

# 2. API JavaFX
No Java 8, 9 e 10, nenhuma configuração adicional é necessária para começar a trabalhar com a biblioteca JavaFX. O projeto será removido do JDK a partir do JDK 11.

### 2.1. Arquitetura
JavaFX usa pipeline de gráficos acelerados por hardware para a renderização, conhecido como Prism. Além do mais, para acelerar totalmente o uso de gráficos, ele aproveita o mecanismo de renderização de software ou hardware, usando internamente DirectX e OpenGL.

O JavaFX tem uma camada de kit de ferramentas de janelas Glass dependente da plataforma para se conectar ao sistema operacional nativo. Ele usa a fila de eventos do sistema operacional para agendar o uso do thread. Além disso, ele lida de forma assíncrona com janelas, eventos, temporizadores.

Os mecanismos de mídia e da Web permitem a reprodução de mídia e suporte a HTML / CSS.

Aqui, notamos dois contêineres principais:

- O palco é o contêiner principal e o ponto de entrada do aplicativo. Ele representa a janela principal e é passado como um argumento do método start().
- Scene é um contêiner para conter os elementos da IU, como visualizações de imagens, botões, grades, caixas de texto.

A cena pode ser substituída ou alternada para outra cena. Isso representa um gráfico de objetos hierárquicos, conhecido como Scene Graph. Cada elemento nessa hierarquia é chamado de nó. Um único nó tem seu ID, estilo, efeitos, manipuladores de eventos, estado.

Além disso, a cena também contém os contêineres de layout, imagens e mídia.

2.2. Tópicos
No nível do sistema, a JVM cria threads separados para executar e renderizar o aplicativo:

- Thread de renderização de prisma - responsável por renderizar o Scene Graph separadamente;
- Thread do aplicativo - é o thread principal de qualquer aplicativo JavaFX. Todos os nós e componentes ativos são anexados a este encadeamento.

### 2.3. Ciclo da vida
A classe javafx.application.Application tem os seguintes métodos de ciclo de vida:

- init () - é chamado depois que a instância do aplicativo é criada. Neste ponto, a API JavaFX ainda não está pronta, então não podemos criar componentes gráficos aqui.
- start (Stage stage) - todos os componentes gráficos são criados aqui. Além disso, o segmento principal para as atividades gráficas começa aqui.
stop () - é chamado antes do encerramento do aplicativo; por exemplo, quando um usuário fecha a janela principal. É útil substituir esse método para alguma limpeza antes do encerramento do aplicativo.
O método estático launch () inicia o aplicativo JavaFX.

### 2.4. FXML
JavaFX usa uma linguagem de marcação FXML especial para criar as interfaces de visualização.

Isso fornece uma estrutura baseada em XML para separar a visualização da lógica de negócios. XML é mais adequado aqui, pois é capaz de representar de forma bastante natural uma hierarquia de Scene Graph.

Finalmente, para carregar o arquivo .fxml, usamos a classe FXMLLoader, que resulta no gráfico de objeto da hierarquia da cena.

# 3. Primeiros passos
Para ser prático, vamos construir um pequeno aplicativo que permite pesquisar em uma lista de pessoas.

Primeiro, vamos adicionar uma classe de modelo Person - para representar nosso domínio:

```
public class Person {
    private SimpleIntegerProperty id;
    private SimpleStringProperty name;
    private SimpleBooleanProperty isEmployed;

    // getters, setters
}
```

Observe como, para encerrar os valores int, String e boolean, estamos usando as classes SimpleIntegerProperty, SimpleStringProperty, SimpleBooleanProperty no pacote javafx.beans.property.

A seguir, vamos criar a classe Main que estende a classe abstrata Application:

```
public class Main extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception {
        FXMLLoader loader = new FXMLLoader(
          Main.class.getResource("/SearchController.fxml"));
        AnchorPane page = (AnchorPane) loader.load();
        Scene scene = new Scene(page);

        primaryStage.setTitle("Title goes here");
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```

Nossa classe principal sobrescreve o método start (), que é o ponto de entrada do programa.

Em seguida, o FXMLLoader carrega a hierarquia do gráfico de objeto de SearchController.fxml para o AnchorPane.

Depois de iniciar uma nova cena, nós a definimos como o palco principal. Também definimos o título de nossa janela e o showit().

Observe que é útil incluir o método main() para poder executar o arquivo JAR sem o JavaFX Launcher.

### 3.1. FXML View
Agora, vamos mergulhar mais fundo no arquivo XML SearchController.

Para nosso aplicativo de pesquisa, adicionaremos um campo de texto para inserir a palavra-chave e o botão de pesquisa:

```
<AnchorPane 
  xmlns:fx="http://javafx.com/fxml"
  xmlns="http://javafx.com/javafx"
  fx:controller="com.baeldung.view.SearchController">
    <children>

        <HBox id="HBox" alignment="CENTER" spacing="5.0">
            <children>
                <Label text="Search Text:"/>
                <TextField fx:id="searchField"/>
                <Button fx:id="searchButton"/>
            </children>
        </HBox>

        <VBox fx:id="dataContainer"
              AnchorPane.leftAnchor="10.0"
              AnchorPane.rightAnchor="10.0"
              AnchorPane.topAnchor="50.0">
        </VBox>

    </children>
</AnchorPane>
```

AnchorPane é o contêiner raiz aqui e o primeiro nó da hierarquia do gráfico. Ao redimensionar a janela, ele reposicionará o filho em seu ponto de ancoragem. O atributo fx: controller conecta a classe Java com a marcação.

Existem alguns outros layouts integrados disponíveis:

- BorderPane - divide o layout em cinco seções: superior, direita, inferior, esquerda, centro
- HBox - organiza os componentes filhos em um painel horizontal
- VBox - os nós filhos são organizados em uma coluna vertical
- GridPane - útil para criar uma grade com linhas e colunas
Em nosso exemplo, dentro do painel HBox horizontal, usamos um Label para colocar o texto, TextField para a entrada e um Button. Com fx: id marcamos os elementos para que possamos usá-los posteriormente no código Java.

O painel VBox é onde exibiremos os resultados da pesquisa.

Então, para mapeá-los para os campos Java - usamos a anotação @FXML:

```
public class SearchController {
 
    @FXML
    private TextField searchField;
    @FXML
    private Button searchButton;
    @FXML
    private VBox dataContainer;
    @FXML
    private TableView tableView;
    
    @FXML
    private void initialize() {
        // search panel
        searchButton.setText("Search");
        searchButton.setOnAction(event -> loadData());
        searchButton.setStyle("-fx-background-color: #457ecd; -fx-text-fill: #ffffff;");

        initTable();
    }
}
```

Depois de preencher os campos anotados com @FXML, initialize() será chamado automaticamente. Aqui, podemos realizar outras ações sobre os componentes da IU - como registrar ouvintes de eventos, adicionar estilo ou alterar a propriedade do texto.

No método initTable (), criaremos a tabela que conterá os resultados, com 3 colunas, e a adicionaremos ao VBox dataContainer:

```
private void initTable() {        
    tableView = new TableView<>();
    TableColumn id = new TableColumn("ID");
    TableColumn name = new TableColumn("NAME");

    TableColumn employed = new TableColumn("EMPLOYED");
    tableView.getColumns().addAll(id, name, employed);
    dataContainer.getChildren().add(tableView);
}
```

Finalmente, toda essa lógica descrita aqui produzirá a seguinte janela:

<img src="https://github.com/isaccanedo/Introduction-to-JavaFx/blob/master/Janela.png">

# 4. API de ligação
Agora que os aspectos visuais foram tratados, vamos começar a olhar para os dados de ligação.

A API de ligação fornece algumas interfaces que notificam objetos quando ocorre uma alteração de valor de outro objeto.

Podemos vincular um valor usando o método bind () ou adicionando ouvintes.

A ligação unidirecional fornece uma ligação para apenas uma direção:

´´´
searchLabel.textProperty().bind(searchField.textProperty());
´´´

Aqui, qualquer alteração no campo de pesquisa atualizará o valor do texto do rótulo.

Por comparação, a ligação bidirecional sincroniza os valores de duas propriedades em ambas as direções.

A forma alternativa de vincular os campos são ChangeListeners:

```
searchField.textProperty().addListener((observable, oldValue, newValue) -> {
    searchLabel.setText(newValue);
});
```

A interface Observable permite observar o valor do objeto para alterações.

Para exemplificar isso, a implementação mais comumente usada é a interface javafx.collections.ObservableList <T>:

```
ObservableList<Person> masterData = FXCollections.observableArrayList();
ObservableList<Person> results = FXCollections.observableList(masterData);
```

Aqui, qualquer alteração do modelo, como inserção, atualização ou remoção dos elementos, notificará os controles da IU imediatamente.

A lista masterData conterá a lista inicial de objetos Person e a lista de resultados será a lista que exibimos na pesquisa.

Também temos que atualizar o método initTable() para vincular os dados da tabela à lista inicial e conectar cada coluna aos campos da classe Person:

```
private void initTable() {        
    tableView = new TableView<>(FXCollections.observableList(masterData));
    TableColumn id = new TableColumn("ID");
    id.setCellValueFactory(new PropertyValueFactory("id"));
    TableColumn name = new TableColumn("NAME");
    name.setCellValueFactory(new PropertyValueFactory("name"));
    TableColumn employed = new TableColumn("EMPLOYED");
    employed.setCellValueFactory(new PropertyValueFactory("isEmployed"));

    tableView.getColumns().addAll(id, name, employed);
    dataContainer.getChildren().add(tableView);
}
```

# 5. Simultaneidade
Trabalhar com os componentes da IU em um gráfico de cena não é seguro para thread, pois ele é acessado apenas a partir do thread do aplicativo. O pacote javafx.concurrent está aqui para ajudar com multithreading.

Vamos ver como podemos realizar a pesquisa de dados no thread de segundo plano:

```
private void loadData() {
    String searchText = searchField.getText();
    Task<ObservableList<Person>> task = new Task<ObservableList<Person>>() {
        @Override
        protected ObservableList<Person> call() throws Exception {
            updateMessage("Loading data");
            return FXCollections.observableArrayList(masterData
                    .stream()
                    .filter(value -> value.getName().toLowerCase().contains(searchText))
                    .collect(Collectors.toList()));
        }
    };
}
```

Aqui, criamos um objeto javafx.concurrent.Task de tarefa única e substituímos o método call().

O método call() é executado inteiramente no thread de segundo plano e retorna o resultado para o thread de aplicativo. Isso significa que qualquer manipulação dos componentes da IU dentro desse método lançará uma exceção de tempo de execução.

No entanto, updateProgress(), updateMessage() podem ser chamados para atualizar itens de thread do aplicativo. Quando o estado da tarefa muda para o estado SUCCEEDED, o manipulador de eventos onSucceeded() é chamado a partir do thread do aplicativo:

```
task.setOnSucceeded(event -> {
    results = task.getValue();
    tableView.setItems(FXCollections.observableList(results));
});
```

No mesmo retorno de chamada, atualizamos os dados de tableView para a nova lista de resultados.

A tarefa é executável, portanto, para iniciá-la, precisamos apenas iniciar um novo thread com o parâmetro de tarefa:

```
Thread th = new Thread(task);
th.setDaemon(true);
th.start();
```

O sinalizador setDaemon (true) indica que o encadeamento será encerrado após o término do trabalho.

# 6. Tratamento de eventos
Podemos descrever um evento como uma ação que pode ser interessante para o aplicativo.

Por exemplo, as ações do usuário, como cliques do mouse, pressionamentos de tecla, redimensionamento de janela, são manipuladas ou notificadas pela classe javafx.event.Event ou qualquer uma de suas subclasses.

Além disso, distinguimos três tipos de eventos:

- InputEvent - todos os tipos de ações de tecla e mouse como KEY_PRESSED, KEY_TYPED, KEY_RELEASED ou MOUSE_PRESSES, MOUSE_RELEASED;
- ActionEvent - representa uma variedade de ações como disparar um botão ou terminar um KeyFrame;
- WindowEvent - WINDOW_SHOWING, WINDOW_SHOWN
Para demonstrar, o fragmento de código abaixo captura o evento de pressionar a tecla Enter sobre o searchField:

```
searchField.setOnKeyPressed(event -> {
    if (event.getCode().equals(KeyCode.ENTER)) {
        loadData();
    }
});
```

# 7. Estilo
Podemos alterar a interface do usuário do aplicativo JavaFX aplicando um design personalizado a ele.

Por padrão, o JavaFX usa modena.css como um recurso CSS para todo o aplicativo. Isso faz parte do jfxrt.jar.

Para substituir o estilo padrão, podemos adicionar uma folha de estilo à cena:

```
scene.getStylesheets().add("/search.css");
```

Também podemos usar o estilo embutido; por exemplo, para definir uma propriedade de estilo para um nó específico:

```
searchButton.setStyle("-fx-background-color: slateblue; -fx-text-fill: white;");
```

# 8. Conclusão

Este breve artigo cobre os fundamentos da API JavaFX. Analisamos a estrutura interna e apresentamos os principais recursos de sua arquitetura, ciclo de vida e componentes.

Como resultado, aprendemos e agora podemos criar um aplicativo GUI simples.





