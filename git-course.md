= id="intro" data-x="3000" data-y="1500" data-scale="30"

#Curso de Git

<http://bit.ly/cursogitabril>

<https://github.com/abril/git-course>

---
= data-x=0 data-y=0

# O que é?

* Criado em 2005 por Linus Torvalds*
* Necessidade, linux-kernel-devs
* Velocidade
* Design simples
* Suporte robusto a desenvolvimento não linear (milhares de branches paralelos)
* Totalmente distribuído
* Capaz de lidar eficientemente com grandes projetos como o kernel do Linux (velocidade e volume de dados)

---
= data-x=0 data-y=0 data-scale=0.01

# O que é?

Linux usava o BitKeeper para versionar o Linux. Em meiados de 2005, vence a licença de uso e Linus cria seu próprio SCM e, ainda em 2005, é usado oficialmente para versionar o código o Linux. Em 2007 é lançado para uso público.

---
= data-x=1000 data-y=0

# Características Básicas

* Controle de versão distribuido (DVCS)
* Gerenciamento de conteúdo e não arquivos
* Branches como unidade de trabalho
* SHA1 para associação e verificação
* Staging index (preparação para *commit*)
<li style='color: #eee'>Não é o subversion</li>

---
= data-x=2000 data-y=0

# Estrutura de Controle

* Diretório `.git` na raíz de cada projeto
* Arquivos de configuração `.git/config`
* Index `.git/index`
* Hooks `.git/hooks`
* Object Database `.git/objects`
* References `.git/refs`

---
= data-x=2000 data-y=0 data-scale=0.01

# Estrutura de Controle

```
.git
|-- COMMIT_EDITMSG  # última mensagem de commit
|-- HEAD            # referência atual do checkout
|-- config          # arquivo de configuração do repositório
|-- description     # arquivo de descrição do repositório
|-- hooks           # diretório com arquivos de gatilhos
|   |-- ...
|-- index           # mudanças para commit
|-- info
|   |-- exclude     # configuração de arquivos p/ ñ versionamento
|   +-- refs        # lista de branches e suas referências
|-- logs
|   |-- ...         # arquivos de log das mudanças
|-- objects
|   |-- info
|   │   +-- packs   # informações sobre pacotes
|   +-- pack
|       |-- ...     # pacotes e indices
|-- packed-refs
|-- refs
    |-- heads
    +-- tags
```

---
= data-x=3000 data-y=0

# Objects Database

Todos os objetos se registram da mesma forma

```
conteudo

header + conteudo

"2e6f9b0d5885b6010f9167787445617f553a735f"

zlib(header + conteudo)

```
`.git/objects/2e/6f9b0d5885b6010f9167787445617f553a735f`

---
= data-x=3000 data-y=0 data-scale=0.01

# Objects Database

```ruby
def put_raw_object(content, type)
  size = content.length.to_s
  LooseStorage.verify_header(type, size)

  header = "#{type} #{size}\0"
  store = header + content

  sha1 = Digest::SHA1.hexdigest(store)
  path = @directory+'/'+sha1[0...2]+'/'+sha1[2..40]

  if !File.exists?(path)
    content = data-scale=2Zlib::Deflate.deflate(store)

    FileUtils.mkdir_p(@directory+'/'+sha1[0...2])
    File.open(path, 'wb') do |f|
      f.write content
    end
  end
  return sha1
end
```

---
= data-x=4000 data-y=0

# Objects Database

* blob - é o menor objeto git, normalmente representa um arquivo.
* tree - contém *trees* e *blobs*
* commit - agrupa o "estado atual" de todos os objetos (*trees* e *blobs*)
* tag - aponta para um *commit*

<table class='objeto'>
    <thead>
        <tr>
            <td>type</td>
            <td>size</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan='2'>data</td>
        </tr>
    </tbody>
</table>

---
= data-x=5000 data-y=0

# Objects Database

## blobs

<div class='list blobs'>
    <ul class='keys'>
        <li>Rakefile</li>
        <li>source.rb</li>
        <li>source_test.rb</li>
    </ul>
    <ul class='values'>
        <li class='blob'>blob: 2e6f9b</li>
        <li class='blob'>blob: 7034fe</li>
        <li class='blob'>blob: 8034f1</li>
    </ul>
</div>

<table class='objeto'>
    <thead>
        <tr>
            <td>blob</td>
            <td>size</td>
        </tr>
    </thead>
    <tbody>
        <tr class='data'>
            <td colspan='2'>file data</td>
        </tr>
    </tbody>
</table>

`2e6f9b0d5885b6010f9167787445617f553a735f`

---
= data-x=6000 data-y=0

# Objects Database

## trees

<div class='list trees'>
    <ul class='keys'>
        <li>/</li>
        <li>Rakefile</li>
        <li>/lib</li>
        <li>source.rb</li>
        <li>/test</li>
        <li>source_test.rb</li>
    </ul>
    <ul class='values tree-container'>
        <li class='tree'>tree: 1a2b3c</li>
        <li class='blob'>blob: 2e6f9b</li>
        <li class='tree'>tree: 2b3c4d</li>
        <li class='blob'>blob: 7034fe</li>
        <li class='tree'>tree: 5b32ad</li>
        <li class='blob'>blob: 8034f1</li>
    </ul>
</div>

<table class='objeto'>
    <thead>
        <tr>
            <td>tree</td>
            <td>size</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan='2'>
                040000 tree 1a2b3c... /<br/>
                100644 blog 2e6f9b... Rakefile<br/>
                ...
            </td>
        </tr>
    </tbody>
</table>

`ob772ec8eb9ae8952c3c1e56a9ffbe49385cc83a`

---
= data-x=7000 data-y=0

# Objects Database

## commits

<div class='list commits'>
    <ul class='keys'>
        <li>&nbsp;</li>
        <li>/</li>
        <li>Rakefile</li>
        <li>/lib</li>
        <li>source.rb</li>
        <li>/test</li>
        <li>source_test.rb</li>
    </ul>
    <ul class='values tree-container'>
        <li class='commit'>commit: 3e4c5d</li>
        <div class="tree-container">
            <li class='tree'>tree: 1a2b3c</li>
            <li class='blob'>blob: 2e6f9b</li>
            <li class='tree'>tree: 2b3c4d</li>
            <li class='blob'>blob: 7034fe</li>
            <li class='tree'>tree: 5b32ad</li>
            <li class='blob'>blob: 8034f1</li>
        </div>
    </ul>
</div>

<table class='objeto'>
    <thead>
        <tr>
            <td>commit</td>
            <td>size</td>
        </tr>
    </thead>
    <tbody>
        <tr class='data'>
            <td colspan='2'>
                tree ob772e...<br/>
                parent 12cb23...<br/>
                author Someone &lt;aliress&gt;<br/>
                commiter Someone &lt;aliress&gt;<br/>
                commit message<br/>
                ...
            </td>
        </tr>
    </tbody>
</table>

`7216b02627bc3d6ef57008f7ff67f0f8f13f488e`

---
= data-x=8000 data-y=0

# Objects Database

## commits

<div class='only-commits'>
    <ul class='keys'>
        <li>&nbsp;</li>
        <li>/</li>
        <li class='file-changed'>Rakefile</li>
        <li>/lib</li>
        <li>source.rb</li>
        <li>/test</li>
        <li class='file-changed'>source_test.rb</li>
    </ul>
    <ul class='values tree-container'>
        <li class='commit'>commit: 3e4c5d</li>
        <div class="tree-container">
            <li class='tree'>tree: 1a2b3c</li>
            <li class='blob'>blob: 2e6f9b</li>
            <li class='tree'>tree: 2b3c4d</li>
            <li class='blob'>blob: 7034fe</li>
            <li class='tree'>tree: 5b32ad</li>
            <li class='blob'>blob: 8034f1</li>
        </div>
    </ul>
    <ul class='values tree-container'>
        <li class='commit'>commit: 61db26</li>
        <div class="tree-container">
            <li class='tree'>tree: 1a2b3c</li>
            <li class='blob-changed'>blob: 41d300</li>
            <li class='tree'>tree: 2b3c4d</li>
            <li class='blob'>blob: 7034fe</li>
            <li class='tree'>tree: 5b32ad</li>
            <li class='blob-changed'>blob: 2a03fb</li>
        </div>
    </ul>
</div>

---
= data-x=9000 data-y=0

# Objects Database

## tags

<div class='list tags'>
    <ul class='keys'>
        <li>&nbsp;</li>
        <li>&nbsp;</li>
        <li>/</li>
        <li>Rakefile</li>
        <li>/lib</li>
        <li>source.rb</li>
        <li>/test</li>
        <li>source_test.rb</li>
    </ul>
    <ul class='values'>
        <li class='tag'>tag: 6e7a8b</li>
        <div class="tree-container">
            <li class='commit'>commit: 3e4c5d</li>
            <div class="tree-container">
                <li class='tree'>tree: 1a2b3c</li>
                <li class='blob'>blob: 2e6f9b</li>
                <li class='tree'>tree: 2b3c4d</li>
                <li class='blob'>blob: 7034fe</li>
                <li class='tree'>tree: 5b32ad</li>
                <li class='blob'>blob: 8034f1</li>
            </div>
        </div>
    </ul>
</div>

<table class='objeto'>
    <thead>
        <tr>
            <td>tag</td>
            <td>size</td>
        </tr>
    </thead>
    <tbody>
        <tr class='data'>
            <td colspan='2'>
                object 7216b0...<br/>
                tag tag-name<br/>
                ...
            </td>
        </tr>
    </tbody>
</table>

`0bc9a42eb66d7ae36bf44af8ff5a3888e8a02d12`

---
= data-x=10000 data-y=0

# Instalação

Acessar o site oficial do Git e fazer o download de acordo com seu sistema operacional.

<http://git-scm.com/downloads>

---
= data-x=0 data-y=1000

# Configurando

Sintaxe:

```bash
$ git config [&lt;options&gt;] [&lt;key&gt; &lt;value&gt;]
```

1. Sistema `--system`: `/etc/gitconfig`

    ```bash
    $ git config --system
    ```

2. Global `--global`: `~/.gitconfig`

    ```bash
    $ git config --global user.name "Celestino Gomes"
    $ git config --global user.email "celestino.gomes@abril.com.br"
    ```

3. Repositório: `./.git/config`

    ```bash
    $ git config core.ignorecase true
    ```

```bash
$ git config user.name   # Imprime "Celestino Gomes"
$ git config --list      # Lista todas as chaves configuradas para o repo atual
```

---
= data-x=1000 data-y=1000

# Obtendo ajuda

```bash
$ git help

usage: git &lt;command&gt; [&lt;args&gt;]

The most commonly used git commands are:
   add        Add file contents to the index
   bisect     Find by binary search the change that introduced a bug
   branch     List, create, or delete branches
   checkout   Checkout a branch or paths to the working tree
   clone      Clone a repository into a new directory
   ...
   tag        Create, list, delete or verify a tag object signed with GPG

See 'git help &lt;command&gt;' for more information on a specific command.
```

---
= data-x=2000 data-y=1000

# O Básico

1. `git init`: Criando um repositório a partir do diretório atual
