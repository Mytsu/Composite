# Padrões de Projeto - Composite
##### Instituto Federal de Minas Gerais - Campus Formiga

___

### Problema

Imagine que você está fazendo um sistema de gerenciamento de arquivos. Como você já sabe é possível criar arquivos concretos (vídeos, textos, imagens, etc.) e arquivos pastas, que armazenam outros arquivos. O problema é o mesmo, como fazer um design que atenda estes requerimentos?

### Uma Solução

Podemos criar uma classe que representa arquivos que são Pastas, estas pastas teriam uma lista de arquivos concretos e uma lista de arquivos de pastas. Então poderíamos adicionar pastas e arquivos em uma pasta e, a partir dela navegar pelas suas pastas e seus arquivos.

![Composite1](http://i.imgur.com/kuA2bpa.png)

O problema com este design é a interface que esta classe deverá ter. Como são duas listas diferentes precisaríamos de métodos específicos para tratar cada uma dela, ou seja, um método para inserir arquivos e outro para inserir pastas, um método para excluir pastas e outro para excluir arquivos, e assim vai.

Sempre que quisermos inserir uma nova funcionalidade no gerenciador precisaremos criar a mesma funcionalidade para arquivos e pastas. Além disso, sempre que quisermos percorrer uma pasta será necessário percorrer as duas listas, mesmo que vazias.

Bom, vamos pensar em outra solução. E se utilizássemos uma classe base Arquivo para todos os arquivos, assim precisaríamos apenas de uma lista e de um conjunto de funções. Vejamos abaixo:

![Composite2](http://i.imgur.com/Ir1ujyy.png)

Pronto, resolvido o problema dos métodos duplicados. E agora, será que está tudo bem? Como faríamos a diferenciação entre um Arquivo e uma Pasta? Poderíamos utilizar o “instace of” e verificar qual o tipo do objeto, o problema é que seria necessário fazer isso SEMPRE, pois não poderíamos confiar que, dado um objeto qualquer, ele é um arquivo ou uma pasta! Sempre teríamos que fazer isso:

```Java

if(arquivo instanceof Arquivo){ 
        // Codigo para tratar arquivos de video 
    } else if(arquivo instanceof Pasta){ 
        // Codigo para tratar arquivos de audio 
    }

```

Ok, então vamos ver uma boa solução para o problema: o padrão Composite!

___

### Composite

Como o nosso problema era uniformizar o acesso aos arquivos e pastas, provavelmente o Composite seja uma boa solução.

A ideia do Composite é criar uma classe base que contém toda a interface necessária para todos os elementos e criar um elemento especial que agrega outros elementos. Vamos trazer para o nosso exemplo inicial para tentar esclarecer:

![Composite3](http://i.imgur.com/n2fMiYn.png)

A classe base Arquivo implementa todos os métodos necessários para arquivos e pastas, no entanto considera como implementação padrão a do arquivo, ou seja, caso o usuário tente inserir um arquivo em outro arquivo uma exceção será disparada. Veja o código da classe abaixo:

```Java

public abstract class ArquivoComponent {
 
    String nomeDoArquivo;
 
    public void printNomeDoArquivo() {
        System.out.println(this.nomeDoArquivo);
    }
 
    public String getNomeDoArquivo() {
        return this.nomeDoArquivo;
    }
 
    public void adicionar(ArquivoComponent novoArquivo) throws Exception {
        throw new Exception("Nao pode inserir arquivos em: "
                + this.nomeDoArquivo + " - Nao e uma pasta");
    }

    public void remover(String nomeDoArquivo) throws Exception {
        throw new Exception("Nao pode remover arquivos em: "
                + this.nomeDoArquivo + " -Nao e uma pasta");
    }
 
    public ArquivoComponent getArquivo(String nomeDoArquivo) throws Exception {
        throw new Exception("Nao pode pesquisar arquivos em: "
                + this.nomeDoArquivo + " - Nao e uma pasta");
    }
}

```

Uma vez que tudo foi definido nesta classe, para criar um arquivo de vídeo por exemplo, basta implementar o construtor:

```Java

public class ArquivoVideo extends ArquivoComponent {
 
    public ArquivoVideo(String nomeDoArquivo) {
        this.nomeDoArquivo = nomeDoArquivo;
    }
}

```

Já na classe que representa a Pasta nós sobrescrevemos o comportamento padrão e repassamos a chamada para todos os arquivos, sejam arquivos ou pastas, como podemos ver a seguir:

```Java

public class ArquivoComposite extends ArquivoComponent {
 
    ArrayList<ArquivoComponent> arquivos = new ArrayList<ArquivoComponent>();
 
    public ArquivoComposite(String nomeDoArquivo) {
        this.nomeDoArquivo = nomeDoArquivo;
    }
 
    @Override
    public void printNomeDoArquivo() {
        System.out.println(this.nomeDoArquivo);
        for (ArquivoComponent arquivoTmp : arquivos) {
            arquivoTmp.printNomeDoArquivo();
        }
    }
 
    @Override
    public void adicionar(ArquivoComponent novoArquivo) {
        this.arquivos.add(novoArquivo);
    }
 
    @Override
    public void remover(String nomeDoArquivo) throws Exception {
        for (ArquivoComponent arquivoTmp : arquivos) {
            if (arquivoTmp.getNomeDoArquivo() == nomeDoArquivo) {
                this.arquivos.remove(arquivoTmp);
                return;
            }
        }
        throw new Exception("Nao existe este arquivo");
    }

    @Override
    public ArquivoComponent getArquivo(String nomeDoArquivo) throws Exception {
        for (ArquivoComponent arquivoTmp : arquivos) {
            if (arquivoTmp.getNomeDoArquivo() == nomeDoArquivo) {
                return arquivoTmp;
            }
        }
        throw new Exception("Nao existe este arquivo");
    }
 
}

```

Com isto não é necessário conhecer a implementação dos objetos concretos, muito menos fazer cast. Veja como poderíamos utilizar o código do Composite:

```Java

public static void main(String[] args) {
    ArquivoComponent minhaPasta = new ArquivoComposite("Minha Pasta/");
    ArquivoComponent meuVideo = new ArquivoVideo("meu video.avi");
    ArquivoComponent meuOutroVideo = new ArquivoVideo("serieS01E01.mkv");
 
    try {
        meuVideo.adicionar(meuOutroVideo);
    } catch (Exception e) {
        System.out.println(e.getMessage());
    }
 
    try {
        minhaPasta.adicionar(meuVideo);
        minhaPasta.adicionar(meuOutroVideo);
        minhaPasta.printNomeDoArquivo();
    } catch (Exception e) {
        System.out.println(e.getMessage());
    }
 
    try {
        System.out.println("\nPesquisando arquivos:");
        minhaPasta.getArquivo(meuVideo.getNomeDoArquivo())
                .printNomeDoArquivo();
        System.out.println("\nRemover arquivos");
        minhaPasta.remover(meuVideo.getNomeDoArquivo());
        minhaPasta.printNomeDoArquivo();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

```

Agora podemos visualizar a tal estrutura de árvore, suponha que temos pastas dentro de pastas com arquivos, a estrutura seria parecida com a de uma árvore, veja a seguir:

![Composite4](http://i.imgur.com/0xdpoZj.jpg)

Como uma estrutura de árvore temoas Nós e Folhas. No padrão Composite os arquivos concretos do nosso exemplo são chamados de Folhas, pois não possuem filhos e os arquivos pasta são chamados de Nós, pois possuem filhos e fornecem operações sobre esses filhos.

___

### Conclusão

A primeira vantagem, e talvez a mais forte, seja o fato de os clientes do código Composite serem bem simplificados, pois podem tratar todos os objetos da mesma maneira. No nosso exemplo utilizei exceções para que ficasse mais evidente quando um método de uma Pasta é chamado em um Arquivo, mas suponha que os métodos não fizessem nada, a utilização seria mais simplificada ainda, pois não precisaríamos de blocos try e catch.

No entanto, o mal tratamento destas exceções podem gerar problemas de segurança e ai surge uma outra forma de implementar o padrão, restringindo a interface comum dos objetos. Para isto basta remover os métodos de gerenciamento de arquivos (adicionar, remover, etc) da classe base, assim apenas os arquivos pastas teriam estes métodos.

Em contrapartida o usuário do código precisa ter certeza se um dado objeto é Pasta para realizar um cast e chamar os métodos da pasta. 

___

### Referências

> GAMMA, Erich et al. Padrões de Projeto: Soluções reutilizáveis de software orientado a objetos.

___

##### Feito por:
> Mão na massa: Composite https://brizeno.wordpress.com/category/padroes-de-projeto/composite/