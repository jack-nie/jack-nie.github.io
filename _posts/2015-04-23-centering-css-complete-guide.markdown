---
layout: post
title:  "CSS居中完全指导手册"
date:   2015-04-23
---

前端开发中，CSS居中一直是一个令人头痛的问题。为什么CSS居中这么困难呢？其实实现CSS居中并不难，难的是如何从众多的CSS居中方法中选择一个合适的解决方案，通常你并不知道那种解决方案适合当前的情形。所以，让我们来整理一个决策树，希望能够帮助大家更容易的解决CSS居中的问题。
###  一.水平居中
####  1.是否是 inline或者inline-*元素
  将inline元素放置在block父元素中，然后给父元素加上`text-align: center;`

        {% highlight html %}
    HTML: 
    <header>
      This text is centered.
    </header>
    
    <nav role='navigation'>
      <a href="#0">One</a>
      <a href="#0">Two</a>
      <a href="#0">Three</a>
      <a href="#0">Four</a>
    </nav>
    {% endhighlight %}

    {% highlight css%}
CSS:
    body {
      background: #f06d06;
    }
    
    header, nav {
      text-align: center;
      background: white;
      margin: 20px 0;
      padding: 10px;
    }
   
    nav a {
      text-decoration: none;
      background: #333;
      border-radius: 5px;
      color: white;
      padding: 3px 8px;
    }
    {% endhighlight %}
####  2.是否是block元素
  可以利用`margin:0 auto;`来实现水平居中，利用这种技术的前提是该元素拥有一个固定的宽度。
    {% highlight html %}
HTML:
     <main>
      <div class="center">
     I'm a block level element and am centered.
      </div>
     </main>
        {% endhighlight %}
    {% highlight css %}
CSS:
    body {
      background: #f06d06;
    }
    
    main {
      background: white;
      margin: 20px 0;
      padding: 10px;
    }
    
    .center {
      margin: 0 auto;
      width: 200px;
      background: black;
      padding: 20px;
      color: white;
    }
        {% endhighlight %}
####  3.是否有多个block元素需要居中
  如果你有多种block级别的元素需要在一行中水平居中，那么你最好赋予这些元素的display属性不同的值。

    {% highlight html %}
HTML:
    <main class="inline-block-center">
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
      </div>
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row. I have more content in me than my siblings do.
      </div>
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
      </div>
    </main>
    <main class="flex-center">
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
      </div>
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row. I have more content in me than my siblings do.
      </div>
      <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
      </div>
    </main>
        {% endhighlight %}
    {% highlight css %}
CSS:
    body {
      background: #f06d06;
      font-size: 80%;
    }
    
    main {
      background: white;
      margin: 20px 0;
      padding: 10px;
    }
    
    main div {
      background: black;
      color: white;
      padding: 15px;
      max-width: 125px;
      margin: 5px;
    }
    
    .inline-block-center {
      text-align: center;
    }
    .inline-block-center div {
      display: inline-block;
      text-align: left;
    }
    
    .flex-center {
      display: flex;
      justify-content: center;
    }
        {% endhighlight%}
###二.垂直居中
相对而言，CSS垂直居中的实现更加的复杂一点。
####1.是否是inline或者inline-*元素

- 需要居中的元素是否只有一行?
  有时候赋予元素相等的上下内边距就可以实现元素的垂直居中。
    {% highlight html %}
HTML:
        <main>
          <a href="#0">We're</a>
          <a href="#0">Centered</a>
          <a href="#0">Bits of</a>
          <a href="#0">Text</a>
        </main>
        {% endhighlight %}        
    {% highlight css %}
CSS:
        body {
          background: #f06d06;
          font-size: 80%;
        }
    
        main {
          background: white;
          margin: 20px 0;
          padding: 50px;
        }
    
        main a {
          background: black;
          color: white;
          padding: 40px 30px;
          text-decoration: none;
        }
        {% endhighlight %}
  如果padding的解决方案不能满足要求，并且需要居中的文字不会被包裹，那么可以设置line-height和height的值相等。

    {% highlight html %}
HTML:
        <main>
          <div>
              I'm a centered line.
         </div>
        </main>
        {% endhighlight %}        
    {% highlight css %}
CSS:
        body {
          background: #f06d06;
          font-size: 80%;
        }
     
        main {
          background: white;
          margin: 20px 0;
          padding: 40px;
        }
    
        main div {
          background: black;
          color: white;
          height: 100px;
          line-height: 100px;
          padding: 20px;
          width: 50%;
          white-space: nowrap;
        }
        {% endhighlight %}    



- 需要居中的元素是否有多行？
  设置相同的上下内边距同样可以让多行的inline元素居中，如果这种技术不能够实现居中，可以采用`display:table-cell;`来实现垂直居中。
    {% highlight html %}
HTML:
          <table>
            <tr>
              <td>
                I'm vertically centered multiple lines of text in a real table cell.
             </td>
            </tr>
          </table>
      
          <div class="center-table">
            <p>I'm vertically centered multiple lines of text in a CSS-created table layout.</p>
          </div>
        {% endhighlight %}          
    {% highlight css %}
CSS:
          body {
            background: #f06d06;
            font-size: 80%;
          }
      
          table {
            background: white;
            width: 240px;
            border-collapse: separate;
            margin: 20px;
            height: 250px;
          }
      
          table td {
            background: black;
            color: white;
            padding: 20px;
            border: 10px solid white;
            /* default is vertical-align: middle; */
          }
      
          .center-table {
            display: table;
            height: 250px;
            background: white;
            width: 240px;
            margin: 20px;
          }
          .center-table p {
            display: table-cell;
            margin: 0;
            background: black;
            color: white;
            padding: 20px;
            border: 10px solid white;
            vertical-align: middle;
          }
        {% endhighlight %}
  另外的一种实现垂直居中的方案是采用`display:flex;`这种方案相对来讲也比较简单。利用flex-box方案需要父容器有一个固定的高度。
    {% highlight html %}
HTML:
          <div class="flex-center">
            <p>I'm vertically centered multiple lines of text in a flexbox container.</p>
          </div>
        {% endhighlight %}
    {% highlight css %}
CSS:      
          body {
              background: #f06d06;
              font-size: 80%;
          }
        
          div {
              background: white;
              width: 240px;
              margin: 20px;
           }
        
          .flex-center {
              background: black;
              color: white;
              border: 10px solid white;
              display: flex;
              flex-direction: column;
              justify-content: center;
              height: 200px;
              resize: vertical;
              overflow: auto;
          }
          .flex-center p {
              margin: 0;
              padding: 20px;
          }
        {% endhighlight %}
  如果以上方案都不能满足要求，可以尝试采用"Goast Element"技术，该技术在容器中放置一个100%高度的伪元素来实现目标元素的居中。
    {% highlight html %}
HTML:
        <div class="ghost-center">
          <p>I'm vertically centered multiple lines of text in a container. Centered with a ghost pseudo element</p>
        </div>
        {% endhighlight %}
    {% highlight css %}
CSS:      
        body {
          background: #f06d06;
          font-size: 80%;
        }
        
        div {
          background: white;
          width: 240px;
          height: 200px;
          margin: 20px;
          color: white;
          resize: vertical;
          overflow: auto;
          padding: 20px;
        }
        
        .ghost-center {
          position: relative;
        }
        .ghost-center::before {
          content: " ";
          display: inline-block;
          height: 100%;
          width: 1%;
          vertical-align: middle;
        }
        .ghost-center p {
          display: inline-block;
          vertical-align: middle;
          width: 190px;
          margin: 0;
          padding: 20px;
          background: black;
        }
        {% endhighlight %}
####2.是否是block元素
 

- 元素高度已知
 大多数情况下我们并不知道需要居中的元素的高度，原因有很多，这里不赘述。倘若你知道元素的高度，那么可以利用如下的方法实现block元素的垂直居中。

    {% highlight html %}
HTML:
        <main>
      
          <div>
             I'm a block-level element with a fixed height, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight %}
    {% highlight css %}
CSS:        
        body {
          background: #f06d06;
          font-size: 80%;
        }
        
        main {
          background: white;
          height: 300px;
          margin: 20px;
          width: 300px;
          position: relative;
          resize: vertical;
          overflow: auto;
        }
        
        main div {
          position: absolute;
          top: 50%;
          left: 20px;
          right: 20px;
          height: 100px;
          margin-top: -70px;
          background: black;
          color: white;
          padding: 20px;
        }
        {% endhighlight  %}
- 元素的高度未知
    
    {% highlight html %}
HTML:    
        <main>
          
          <div>
             I'm a block-level element with an unknown height, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight %}
    {% highlight css %}
CSS:
        body {
          background: #f06d06;
          font-size: 80%;
        }
        
        main {
          background: white;
          height: 300px;
          margin: 20px;
          width: 300px;
          position: relative;
          resize: vertical;
          overflow: auto;
        }
        
        main div {
          position: absolute;
          top: 50%;
          left: 20px;
          right: 20px;
          background: black;
          color: white;
          padding: 20px;
          transform: translateY(-50%);
          resize: vertical;
          overflow: auto;
        }
        {% endhighlight %}
#### 3.是否可以使用flex-box
    {% highlight html %}
HTML:
        <main>
          
          <div>
             I'm a block-level element with an unknown height, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight %} 
    {% highlight css %}
CSS:      
        body {
          background: #f06d06;
          font-size: 80%;
        }
        
        main {
          background: white;
          height: 300px;
          width: 200px;
          padding: 20px;
          margin: 20px;
          display: flex;
          flex-direction: column;
          justify-content: center;
          resize: vertical;
          overflow: auto;
        }
        
        main div {
          background: black;
          color: white;
          padding: 20px;
          resize: vertical;
          overflow: auto;
        }
        {% endhighlight %}       
### 三.水平居中和垂直居中
 可以组合前文提到的方案来实现元素的水平和垂直居中，下面给出三种比较常见的组合。
#### 1.元素的宽度和高度固定
 使用负边距来定位，左右负边距设置为宽度的一半，上下负边距设定为高度的一半，这种方案的兼容性很好。

    {% highlight html %}
HTML:    
        <main>
          
          <div>
             I'm a block-level element a fixed height and width, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight%}
    {% highlight css %}
CSS:        
        body {
          background: #f06d06;
          font-size: 80%;
          padding: 20px;
        }
        
        main {
          position: relative;
          background: white;
          height: 200px;
          width: 60%;
          margin: 0 auto;
          padding: 20px;
          resize: both;
          overflow: auto;
        }
        
        main div {
          background: black;
          color: white;
          width: 200px;
          height: 100px;
          margin: -70px 0 0 -120px;
          position: absolute;
          top: 50%;
          left: 50%;
          padding: 20px;
        }
        {% endhighlight %}
####2.元素的宽度和高度未知。
 如果元素的高度和宽度未知，那么可以利用transform属性来实现。

    {% highlight html %}
HTML:
        <main>
          
          <div>
             I'm a block-level element of an unknown height and width, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight %}
    {% highlight css %}
CSS:        
        body {
          background: #f06d06;
          font-size: 80%;
          padding: 20px;
        }
        
        main {
          position: relative;
          background: white;
          height: 200px;
          width: 60%;
          margin: 0 auto;
          padding: 20px;
          resize: both;
          overflow: auto;
        }
        
        main div {
          background: black;
          color: white;
          width: 50%;
          transform: translate(-50%, -50%);
          position: absolute;
          top: 50%;
          left: 50%;
          padding: 20px;
          resize: both;
          overflow: auto;
        }
        {% endhighlight %}        
#### 3.是否可以使用flex-box

    {% highlight html %}
HTML:
        <main>
          
          <div>
             I'm a block-level-ish element of an unknown width and height, centered vertically within my parent.
          </div>
          
        </main>
        {% endhighlight %}
    {% highlight css %}
CSS:
        body {
          background: #f06d06;
          font-size: 80%;
          padding: 20px;
        }
        
        main {
          background: white;
          height: 200px;
          width: 60%;
          margin: 0 auto;
          padding: 20px;
          display: flex;
          justify-content: center;
          align-items: center;
          resize: both;
          overflow: auto;
        }
        
        main div {
          background: black;
          color: white;
          width: 50%;
          padding: 20px;
          resize: both;
          overflow: auto;
        }
        {% endhighlight %}
参考：[https://css-tricks.com/centering-css-complete-guide/](https://css-tricks.com/centering-css-complete-guide/ "Centering CSS Complete Guide")


