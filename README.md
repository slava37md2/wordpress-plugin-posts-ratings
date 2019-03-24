# wordpress-plugin-posts-ratings
Plugin adds possibility to vote for posts.

Плагин написан всего за 2 дня и вмещает всего 140 строк. 
При активации плагина в таблицу comments добавляются поля pluses и minuses. 
Но так как wordpress не позволяет добавлять в свои таблицы дополнительные поля, то пришлось добавлять их через mysqli.
Поэтому нужно в функции create_fields_in_database_table() в строчке 
$db = mysqli_connect("127.0.0.1", "db_user", "db_user_password", "db_name"); 
ввести свои значения "db_user", "db_user_password", "db_name".

Потом в строчках
$chkcol = mysqli_query($db, "SELECT * FROM `$wp_table` LIMIT 1");
	$mycol = mysqli_fetch_array($chkcol);
	if(!isset($mycol['pluses']))
  
проверяем существуют ли поля pluses и minuses, и если не существуют, то добавляем их
$result = mysqli_query($db, "ALTER TABLE $wp_table ADD pluses smallint UNSIGNED NOT NULL DEFAULT 0") or die(mysqli_error($db));//smallint - 2 байта

В функции my_the_content_filter добавляем рейтинг поста прямо в текст поста
add_filter('the_content', 'my_the_content_filter');//добавляем рейтинг поста
function my_the_content_filter($content)

В строчках 
  $row = $wpdb->get_row( "SELECT * FROM $wpdb->posts WHERE ID=$id" );
	$rating = $row->pluses - $row->minuses;
извлекаем из базы данных количество плюсов и минусов, проголосованных за этот пост.

Если рейтинг положительный, то он будет отображаться зелёным цветом, если отрицательным, то красным.
  if( $rating > 0 ){$rating='+'.$rating;$color='green';}
	if( $rating < 0 ){$color='red';}

Потом добавляем текст с просьбой оценить пост и собственно сам рейтинг
  
При нажатии на стрелку вверх или стрелку вниз, вызывается функция ajax_plus_ili_minus. В неё передаётся 1 или -1 и номер поста.

Функция ajax_plus_ili_minus добавляется через action wp_print_footer_scripts, 
add_action( 'wp_print_footer_scripts', 'hook_javascript' );
function hook_javascript(){

В функции ajax_plus_ili_minus вызывается файл save-rating-vote.php, в котором плюс или минус добавляется в базу данных.

В случае успешного айакса в строчках
document.getElementById('strelka_vverh_posts'+post_id).innerHTML = '';//убираем стрелки
document.getElementById('strelka_vniz_posts'+post_id).innerHTML = '';
стрелки для голосования убираются,

и рейтинг увеличивается или уменьшается:
if(plus_ili_minus==1)document.getElementById('rating'+post_id).innerHTML = parseInt(document.getElementById('rating'+post_id).innerHTML, 10)+1;// изменяем рейтинг
if(plus_ili_minus==-1)document.getElementById('rating'+post_id).innerHTML = parseInt(document.getElementById('rating'+post_id).innerHTML, 10)-1;

Есть сохранение в localStorage. Можно повторно голосовать только через неделю. Перезагрузив страницу, клиент не сможет проголосовать снова.
