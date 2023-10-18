#  Flappy Bird JavaScript Game

<p>Este projeto é uma implementação simplificada do jogo Flappy Bird em JavaScript. 
   Ele inclui funções e classes para criar o ambiente de jogo, o pássaro, as barreiras e a lógica do jogo..</p>

## Requisitos

<p>Para executar este projeto, você precisará de um navegador da web com suporte a JavaScript.</p>

## Como jogar

1. Abra o arquivo HTML index.html em um navegador da web.
2. Pressione a tecla `espaço` para fazer o pássaro voar.
3. Desvie das barreiras para ganhar pontos.
4. O jogo termina quando o pássaro colide com uma barreira.

## Estrutura do Código 

## Função `novoElemento(tagName, className)`

```javascript
function novoElemento(tagName, className) {
    const elem = document.createElement(tagName)
    elem.className = className
    return elem
}
```
<p>Esta função cria um novo elemento HTML com a tag especificada e a classe especificada e retorna o elemento.</p>

## Classe `Barreira(reversa)`
```javascript
function Barreira(reversa = false) {
    this.elemento = novoElemento('div', 'barreira')

    const borda = novoElemento('div', 'borda')
    const corpo = novoElemento('div', 'corpo')
    this.elemento.appendChild(reversa ? corpo : borda)
    this.elemento.appendChild(reversa ? borda : corpo)

    this.setAltura = altura => corpo.style.height = `${altura}px`
}

```
<p>Esta classe representa uma barreira no jogo. Ela possui métodos para definir a altura da barreira e criar a estrutura da mesma.</p>

## Classe `ParDeBarreiras(altura, abertura, x)`
```javascript
function ParDeBarreiras(altura, abertura, x) {
    this.elemento = novoElemento('div', 'par-de-barreiras')

    this.superior = new Barreira(true)
    this.inferior = new Barreira(false)

    this.elemento.appendChild(this.superior.elemento)
    this.elemento.appendChild(this.inferior.elemento)

    this.sortearAbertura = () => {
        const alturaSuperior = Math.random() * (altura - abertura)
        const alturaInferior = altura - abertura - alturaSuperior
        this.superior.setAltura(alturaSuperior)
        this.inferior.setAltura(alturaInferior)
    }

    this.getX = () => parseInt(this.elemento.style.left.split('px')[0])
    this.setX = x => this.elemento.style.left = `${x}px`
    this.getLargura = () => this.elemento.clientWidth

    this.sortearAbertura()
    this.setX(x)
}
```

<p>Esta classe representa um par de barreiras no jogo. Ela cria uma barreira superior e uma inferior e 
   possui métodos para sortear a abertura e definir a posição.</p>

   ## Classe `Barreiras(altura, largura, abertura, espaco, notificarPonto)`
   ```javascript
function Barreiras(altura, largura, abertura, espaco, notificarPonto) {
    this.pares = [
        new ParDeBarreiras(altura, abertura, largura),
        new ParDeBarreiras(altura, abertura, largura + espaco),
        new ParDeBarreiras(altura, abertura, largura + espaco * 2),
        new ParDeBarreiras(altura, abertura, largura + espaco * 3)
    ]

    const deslocamento = 3
    this.animar = () => {
        this.pares.forEach(par => {
            par.setX(par.getX() - deslocamento)

            // quando o elemento sair da área do jogo
            if (par.getX() < -par.getLargura()) {
                par.setX(par.getX() + espaco * this.pares.length)
                par.sortearAbertura()
            }

            const meio = largura / 2
            const cruzouOMeio = par.getX() + deslocamento >= meio
                && par.getX() < meio
            if(cruzouOMeio) notificarPonto()   
        })
    }
}
   ```
<p>Esta classe cria e gerencia as barreiras do jogo. Ela possui um array de pares de barreiras e um método para animá-las. 
   Também notifica quando o jogador ganha pontos.</p>

   ## Classe `Passaro(alturaJogo)`
   ```javascript
    function Passaro(alturaJogo) {
    let voando = false

    this.elemento = novoElemento('img', 'passaro')
    this.elemento.src = 'imgs/passaro.png'

    this.getY = () => parseInt(this.elemento.style.bottom.split('px')[0])
    this.setY = y => this.elemento.style.bottom = `${y}px`

    window.onkeydown = e => voando = true
    window.onkeyup = e => voando = false

    this.animar = () => {
        const novoY = this.getY() + (voando ? 8 : -5)
        const alturaMaxima = alturaJogo - this.elemento.clientHeigth

        if (novoY <= 0) {
            this.setY(0)
        } else if (novoY >= alturaMaxima) {
            this.setY(alturaMaxima)
        } else {
            this.setY(novoY)
        }
    }

    this.setY(alturaJogo / 2)
}

   ```
<p>Esta classe representa o pássaro do jogo. Ela possui métodos para controlar a posição e a animação do pássaro.</p>

## Classe `Progresso()`

```javascript
function Progresso() {
    this.elemento = novoElemento('span', 'progresso')
    this.atualizarPontos = pontos => {
        this.elemento.innerHTML = pontos
    }
    this.atualizarPontos(0)
}
```
<p>Esta classe cria um elemento de progresso para mostrar a pontuação do jogador.</p>

## Função `estaoSobrepostos(elementoA, elementoB)`

```javascript
function estaoSobrepostos(elementoA, elementoB) {
    const a = elementoA.getBoundingClientRect()
    const b = elementoB.getBoundingClientRect()

    const horizontal = a.left + a.width >= b.left
        && b.left + b.width >= a.left
    const vertical = a.top + a.height >= b.top
        && b.top + b.height >= a.top
    return horizontal && vertical        
}
```
<p>Esta função verifica se dois elementos HTML estão sobrepostos.</p>

## Função `colidiu(passaro, barreiras)`

```javascript
function colidiu(passaro, barreiras) {
    let colidiu = false
    barreiras.pares.forEach(ParDeBarreiras => {
        if (!colidiu) {
            const superior = ParDeBarreiras.superior.elemento
            const inferior = ParDeBarreiras.inferior.elemento
            colidiu = estaoSobrepostos(passaro.elemento, superior)
                || estaoSobrepostos(passaro.elemento, inferior)
        }
    })
    return colidiu
}
```
<p>Esta função verifica se o pássaro colidiu com alguma barreira.</p>

## Classe `FlappyBird()`

```javascript
function FlappyBird() {
    let pontos = 0

    const areaDoJogo = document.querySelector('[wm-flappy]')
    const altura = areaDoJogo.clientHeight
    const largura = areaDoJogo.clientWidth

    const progresso = new Progresso()
    const barreiras = new Barreiras(altura, largura, 200, 400,
        () => progresso.atualizarPontos(++pontos))
    const passaro = new Passaro(altura)    

    areaDoJogo.appendChild(progresso.elemento)
    areaDoJogo.appendChild(passaro.elemento)
    barreiras.pares.forEach(par => areaDoJogo.appendChild(par.elemento))

    this.start = () => {
        // loop do jogo
        const temporizador = setInterval(() => {
           barreiras.animar()
           passaro.animar() 

            if (colidiu(passaro, barreiras)) {
                clearInterval(temporizador)
            }
        }, 20)
    }
}
```
<p>Esta classe é responsável por criar o ambiente de jogo, iniciar a animação e controlar o progresso do jogo.</p>

## Executando o Jogo

<p>Para iniciar o jogo, crie uma instância da classe `FlappyBird` e chame o método `start()`.
  O jogo será executado no navegador e você pode jogar pressionando a tecla `espaço` para controlar o pássaro.</p>

```javascript
new FlappyBird().start()
```

## Contribuições

<p>Este é um projeto simples e educativo. Se você deseja contribuir, sinta-se à vontade para melhorar o código, 
  adicionar recursos ou otimizá-lo. Pull requests são bem-vindos.</p>  

## Licença

<p>Este projeto é distribuído sob a licença MIT. Consulte o arquivo `LICENSE` para obter mais informações.

Divirta-se jogando Flappy Bird em JavaScript!</p>


<P> Pra acessar -> https://miltonnn.github.io/projeto-flaapy/</P>
