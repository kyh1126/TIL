# Chapter 6. 마무리

## 1. **파일로 모듈화 - require**

---

- 파일로 코드를 분류해서 정리정돈 하는 방법을 알아본다.
    
    ```php
    <?php
    function print_title(){
      if(isset($_GET['id'])){
        echo $_GET['id'];
      } else {
        echo "Welcome";
      }
    }
    function print_description(){
      if(isset($_GET['id'])){
        echo file_get_contents("data/".$_GET['id']);
      } else {
        echo "Hello, PHP";
      }
    }
    function print_list(){
      $list = scandir('./data');
      $i = 0;
      while($i < count($list)){
        if($list[$i] != '.') {
          if($list[$i] != '..') {
            echo "<li><a href=\"index.php?id=$list[$i]\">$list[$i]</a></li>\n";
          }
        }
        $i = $i + 1;
      }
    }
    ?>
    ```
    
    ```php
    <?php
    require_once('lib/print.php');
    ?>
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>
          <?php
          print_title();
          ?>
        </title>
      </head>
      <body>
        <h1><a href="index.php">WEB</a></h1>
        <ol>
          <?php
          print_list();
          ?>
        </ol>
    ```
    
    ```php
    </body>
    </html>
    ```
    
    ```php
    <?php
    require_once('lib/print.php');
    require_once('view/top.php');
    ?>
        <a href="create.php">create</a>
        <?php if(isset($_GET['id'])) { ?>
          <a href="update.php?id=<?=$_GET['id']?>">update</a>
          <form action="delete_process.php" method="post">
            <input type="hidden" name="id" value="<?=$_GET['id']?>">
            <input type="submit" value="delete">
          </form>
        <?php } ?>
        <h2>
          <?php
          print_title();
          ?>
        </h2>
        <?php
        print_description();
         ?>
    <?php
    require_once('view/bottom.php');
    ?>
    ```
    
    ```php
    <?php
    require('lib/print.php');
    require('view/top.php');
    ?>
        <a href="create.php">create</a>
        <form action="create_process.php" method="post">
          <p>
            <input type="text" name="title" placeholder="Title">
          </p>
          <p>
            <textarea name="description" placeholder="Description"></textarea>
          </p>
          <p>
            <input type="submit">
          </p>
        </form>
    <?php
    require('view/bottom.php');
    ?>
    ```
    
    ```php
    <?php
    require('lib/print.php');
    require('view/top.php');
    ?>
        <a href="create.php">create</a>
        <?php if(isset($_GET['id'])) { ?>
          <a href="update.php?id=<?=$_GET['id']?>">update</a>
        <?php } ?>
        <h2>
         <form action="update_process.php" method="post">
           <input type="hidden" name="old_title" value="<?=$_GET['id']?>">
           <p>
             <input type="text" name="title" placeholder="Title" value="<?php print_title(); ?>">
           </p>
           <p>
             <textarea name="description" placeholder="Description"><?php print_description(); ?></textarea>
           </p>
           <p>
             <input type="submit">
           </p>
         </form>
    <?php
    require('view/bottom.php');
    ?>
    ```
    

## 2. 보안

---

- 웹애플리케이션에게 일어날 수 있는 나쁜 일들을 알아보고, 이런 문제를 해결하는 사례를 알아봅니다.

### ****Cross site scripting (XSS)****

---

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>XSS</title>
  </head>
  <body>
    <h1>Cross site scripting</h1>
    <?php
    echo htmlspecialchars('<script>alert("babo");</script>');
    ?>
  </body>
</html>
```

- `htmlspecialchars` 안붙이면 babo 알럿이 뜬다.

```php
<?php
function print_title(){
  if(isset($_GET['id'])){
    echo htmlspecialchars($_GET['id']);
  } else {
    echo "Welcome";
  }
}
function print_description(){
  if(isset($_GET['id'])){
    echo htmlspecialchars(file_get_contents("data/".$_GET['id']));
  } else {
    echo "Hello, PHP";
  }
}
function print_list(){
  $list = scandir('./data');
  $i = 0;
  while($i < count($list)){
    $title = htmlspecialchars($list[$i]);
    if($list[$i] != '.') {
      if($list[$i] != '..') {
        echo "<li><a href=\"index.php?id=$title\">$title</a></li>\n";
      }
    }
    $i = $i + 1;
  }
}
?>
```

### ****파일 경로 보호****

---

```php
<?php
function print_title(){
  if(isset($_GET['id'])){
    echo htmlspecialchars($_GET['id']);
  } else {
    echo "Welcome";
  }
}
function print_description(){
  if(isset($_GET['id'])){
    $basename = basename($_GET['id']);
    echo htmlspecialchars(file_get_contents("data/".$basename));
  } else {
    echo "Hello, PHP";
  }
}
function print_list(){
  $list = scandir('./data');
  $i = 0;
  while($i < count($list)){
    $title = htmlspecialchars($list[$i]);
    if($list[$i] != '.') {
      if($list[$i] != '..') {
        echo "<li><a href=\"index.php?id=$title\">$title</a></li>\n";
      }
    }
    $i = $i + 1;
  }
}
?>
```

## 3. UI vs API 와 공부방법

---

- API는 애플리케이션을 만들기 위해서 기반이 되는 시스템(웹브라우저, PHP)이 제공하는 기능을 부품으로 사용해야 한다. 이런 기능을 사용하기 위해서 호출하는 명령어를 API라고 한다.
    - PHP에서는 주로 함수의 형태로 API가 제공된다.
- 여기서는 API가 무엇인지 설명하고, 어떤 방법으로 공부하면 좋을지는 필자의 사견을 담아서 이야기해본다.
- UI: User Interface

## 4. 수업을 마치며

---

- 앞으로 겪게 될 문제들을 소개하고, 각 문제를 극복할 수 있도록 도와주는 공부거리를 소개한다.
    - document 읽기
    - php composer: 패키지 매니저
    - federation authentication facebook php 등 키워드를 활용해 사용법 찾아내기