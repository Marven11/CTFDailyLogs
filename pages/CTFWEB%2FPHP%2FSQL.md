# 不安全的使用 SQL 方式
	- ```php
	  <?php
	  
	  $mysqluser="root";
	  $mysqlpwd="root";
	  $mysqldb="sds";
	  $mysqlhost="localhost";
	  $mysqlport="3306";
	  $mysqli=@new mysqli($mysqlhost,$mysqluser,$mysqlpwd,$mysqldb);
	  if(mysqli_connect_errno()){
	  	die(mysqli_connect_error());
	  }
	  
	  $sql="select * from sds_user;";
	  $result=$mysqli->query($sql);
	  $row=$result->fetch_array(MYSQLI_BOTH);
	  var_dump($row);
	  ?>
	  ```
- # 安全的使用SQL方式
	- ```php
	  $stmt = $db->prepare('update name set name = ? where id = ?');
	  $stmt->bind_param('si',$name,$id);
	  $stmt->execute();
	  ```
- # 无密码打Gopher
	- [[CTFWEB/伪协议]]
	- [[CTFWEB/WP/CTFShow/web308]]
-