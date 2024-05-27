![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/3ff95599-09f6-4e5f-b0a8-4fa5b3733cd5)


### Descrição

Essa documentação tem como objetivo descrever como funciona a lógica interna e os cálculos
envolvidos na criação de um gráfico do tipo radar, de modo que posteriormente possam ser
feitas melhorias além de possibilitar rapidamente qualquer um utilizar e modificar o elemento para as necessidades especificas de próximos projetos que venham a utilizar este tipo de estrutura de representação de dados. 
Alguns exemplos de utilização deste gráfico incluem:

- **Análise de Desempenho**:
  - Negócios: Avaliar desempenho de funcionários
- **Pesquisa de Mercado**:
  - Pesquisas de Clientes: Visualizar satisfação
- **Gestão da Qualidade**:
  - Controle de Processos: Monitorar métricas de qualidade
- **Saúde**:
  - Comparação de Dispositivos Médicos: Comparar características de dispositivos
- **Estudos Ambientais**:
  - Avaliação de Ecossistemas: Avaliar indicadores ambientais
  - Métricas de Sustentabilidade: Comparar desempenho de sustentabilidade

---------------------------------

### ==Avisos== 

1. ==O código disponilizado funciona corretamente apenas nas versões 5 ou posteriores devido a introdução da *feature* de backsticks para formatação de *strings* que não está presente nas anteriores== 
2. ==As partes do código colocadas aqui não são a totalidade do código e certos aspectos foram omitidos pela trivialidade do mesmo. Os trechos utilizados são apenas os mais importantes para a compreensão da matemática envolvida.== 

------------------------------

### Descrição da lógica

A principio, deve se pensar: como definir um polígono de lados N ? Para isso é necessário
definir qual será o espaço possível para desenhar este polígono.
Uma possibilidade é estabelecer um quadrado 45 vértices, criando um ponto inicial
comum entre todos os polígonos possíveis.

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/16a811a7-bf9d-4eeb-9301-2179ba001af0)

Quando há um referencia sobe os limites que os polígonos deverão seguir, é possível então definir
o centro de todos, que será o mesmo centro do quadrado que segue a seguinte equação:

$$
x_c = \frac{x_1 + x_2 + x_3 + x_4}{4},
y_c = \frac{y_1 + y_2 + y_3 + y_4}{4}
$$


Definido em **tgml script**: 

```javascript
const center_x = (square_vertices.map((value) => {
        return value[0]
    }).reduce((acc,cv) => {
        return acc + cv}, 0))/4

const center_y = (square_vertices.map((value) => {
	return value[1]
}).reduce((acc,cv) => {
	return acc + cv}, 0))/4
```

Além disso, tendo os pontos é possível definir também o tamanho dos lados do quadrado através da distancia euclidiana: 

$$
d_l = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2} 
$$

Definido em **tgml script**: 

```javascript
const euclideanDistance = (point1,point2) => {
    return Math.sqrt(
        point1
            .map((value, index) => Math.pow(value - point2[index], 2))
            .reduce((acc, curr) => acc + curr, 0)
    );
}
```

Através das laterais do quadrado, torna-se possível definir o ponto inicial em que o polígono de quantidade N de lados irá iniciar seu primeiro vértice, que será o ponto no qual o raio, definido pela equação seguinte, do circulo interior ao quadrado intercepta uma lateral do quadrado. 

$$
r = \frac{d_l}{2}
$$

Definido em **tgml script**

```javascript
const radius = side_length/2 ;
```

Esse raio é definido na metade do lado do quadrado. Se o início for, arbitrariamente, em sentido horário, então o ponto de intersecção se dará:

$$ 
	P = (X1 ,\frac{Y1}{2})
$$

Visualizando geometricamente:

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/67196145-6e63-4121-badb-139fa1797193)


Após definir o início do polígono, é necessário definir onde posicionar os vértices do novo polígono. Para isso basta encontrar a angulação as frações do circulo no qual o polígono está circunscrito utilizando:

$$
a_t = (2*\pi) / N
$$

Definido em **tgml script**:

```javascript
const angle_step = (2 * Math.PI) / n_sides ;
```

Onde $N$ é a quantidade de lados e $`a_t`$ é a angulação interna polígono. Como é visível na imagem seguinte: 

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/e92e7cfb-5b7d-478f-8be2-ca17e6a48491)

Neste caso, $`N = 4`$ e portanto o *passo* da angulação é 90 graus. **Nas próximas equações os resultados serão utilizado em radianos para facilitação das contas.**
Um vez tendo as subdivisões dos ângulos é possível utilizar a equação paramétrica do círculo para encontrar os pontos que estão contidas no perímetro do circulo. Neste caso, o círculo continua sendo o mesmo definido no início. 

$$ 
\begin{align}
v_x = x_c + r \times cos(a_t \times i) \newline
v_y = y_c + r \times sin(a_t \times i)
\end{align}
$$ 

Definido em **tgml script**:

```javascript
var vertices = [] 
for(var i = 0; i <= n_sides ; i++){
        var angle = i * angle_step ; 
        var vertex_x = center_x + radius * Math.cos(angle)
        var vertex_y = center_y + radius * Math.sin(angle)
        
        vertices.push([vertex_x,vertex_y])
    }
```

Visualmente é possível representar como: 

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/3c38aee5-0868-4eb9-8491-4738edb0fa05)

Onde $`V_i = {V_x,V_y}`$   e será iterado a medida em que aumentam a quantidade de lados necessárias para formar a representação gráfica pedida. Até aqui já é possível, operacionalmente, definir os pontos do polígono interno, ou seja aquele que irá representar os dados, porém apenas com seu tamanho máximo ou seja igual ao polígono externo: basta atribuir todos os pontos encontrados ao polígono interno e ele terá o mesmo tamanho. 

```javascript
center.setAttribute("Left",center_x);
    center.setAttribute("Bottom",center_y);
    
    viz_elipse(event.getCurrentTarget(),n_sides)
    vertices.map((vertice,index) => {
        if(index > 0){
            var p = event.getCurrentTarget().getChild(`${index}`);
            p.setAttribute("Left",vertice[0]);
            p.setAttribute("Top",vertice[1]);
        }
    })
```

A ideia, contudo, é utilizar o polígono interno para representar frações do polígono externo de modo que possa representar porcentagens. 
Para representar as informações, é necessário que seja encontrado o ponto exato que representa a porcentagem $`P_r`$ e atribuí-lo ao vértice do polígono interno. Este ponto deve pertencer a mesma reta que intercepta o círculo original no vértice $`V_i`$ . Para encontrar esse ponto basta definir, com base na porcentagem $`P_r`$  escolhida o novo tamanho de uma nova reta sobreposta $`R_n`$:

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/bb7a0948-9f9b-4698-93b0-36ef27fbc4eb)


Dessa forma para encontrar os pontos finais $`n_x, n_y`$, pois os pontos iniciais sempre serão o centro já definido no quadrado no qual o polígono está inscrito, é necessário então descobrir através da equação:

$$
\begin{align}
n_x = x_1 + u_x \times L_n \newline
n_y = y_1 + u_y \times L_n \newline
\end{align}
$$

onde são definidos: 

$$
\begin{align}
u_x =  \frac{\Delta x}{L_o}, 
u_y =  \frac{\Delta y}{L_o}  
\end{align} 
$$

Definido em **tgml script**:

```javascript
const findNewEndpoint = (x1, y1, x2, y2,newLength) => {
        
    const deltaX = x2 - x1;
    const deltaY = y2 - y1;
    
    const originalLength = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
    
    const unitX = deltaX / originalLength;
    const unitY = deltaY / originalLength;
    
    const newX = x1 + unitX * newLength;
    const newY = y1 + unitY * newLength;
    
    return [ newX, newY ];
}
```

$`L_o$`$ é o tamanho original do raio e $`L_n`$ é uma fração dada pela nova porcentagem escolhida ou seja:

$$
L_n = L_o * P_r
$$

Onde, em  **tgml script** pode-se definir a porcentagem e o calculo do novo tamanho como:

```javascript

const calc_length = (x1, y1, x2, y2) => {
    const deltaX = x2 - x1;
    const deltaY = y2 - y1;
    
    const originalLength = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
    return originalLength 
}

const percentage_calc = (length,pos,percentage) => {
    return (percentage/100)*length ; 
}

```

e assim basta iterar e adicionar os pontos ao polígono interno: 

```javascript

const length_ = calc_length(values_.X1,values_.Y1,values_.X2,values_.Y2);
        
const new_length = percentage_calc(length_,parsed,value);
const [X2_new,Y2_new] = findNewEndpoint(values_.X1,values_.Y1,values_.X2,values_.Y2,new_length);
        
var current_v = ((name) => {
	return parseInt(name.split("_")[1])
})(bind_name); 

// definindo os novos pontos através de uma array de pontos 

const points = viz_polygon.getAttribute("Points"); 
const pointsArray = points.split(" "); 
const new_points = pointsArray.map((value,index) => {
	if(index === current_v - 1){
		return `${X2_new},${Y2_new}`
	}else{
		return value 
	}; 
});

// depois basta atribuir esses pontos ao poligono em questão 

...
```

Através disso, portanto é obtido os pontos do polígono interno e assim finalizando o gráfico:

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/f11f1eee-3975-40e4-b830-036171c88f32)


------------------------ 

## Configurações 

Dentro do EBO as seguintes informações são expostas no elemento:

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/0b97785d-e5f4-4d6b-a8d6-3235c9780704)


1. **Polígono Interno**
	1. Opacidade
	2. Cor
	3. Cor Borda 
2. **Aresta**
	1. Opacidade 
	2. Cor
3. **Apótema**
	1. Opacidade 
	2. Cor 
4. **Vértice**
	 1. Opacidade 
	2. Cor

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/66ea5d1a-5ace-4b74-a6c6-6150adba8717)


5. **Opacidade OnOver** 

O valor de **OnOver** é o valor que o polígono irá atingir quando o mouse passar por cima dele. O valor normal de opacidade é aquele definido anteriormente através dos **exposes**.
No exemplo o valor vai de 0.5 para 0.8:


![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/45d10e00-9b69-499c-807a-9a25d261577a)


6. **Número de Informações**

A quantidade máxima de informações, ou seja, de pontos do polígono para representar tais informações, é de 10. As modificações são feitas dinamicamente e os binds escolhidos de acordo com a quantidade de pontos necessárias para representar a quantidade de informações escolhidas. 

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/5e682867-0915-4a7b-921d-c742a1561268)


### Binds 

Para inserir as informações de acordo com o que o usuário quer basta utilizar as binds expostas. Elas leem as informações como porcentagens de 0 a 100%. Como no exemplo a seguir:

![image](https://github.com/synth-me/radar-chart-ebo/assets/68300901/b4d5007d-23f9-461e-9d5e-6d5b692621f0)


----------------------------------------------


