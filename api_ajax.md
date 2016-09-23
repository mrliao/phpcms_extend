```php
header('Content-Type:application/json; charset=utf-8');
// model 配置
$tag = (string)$_GET['tag'];
$page = intval($_GET['page']);
$tag_permission = [
    'content_lists' => ['catid', 'thumb', 'moreinfo', 'pagesize'],
    'content_category' => ['catid', 'thumb', 'moreinfo', 'pagesize']
];

if(array_key_exists($tag, $tag_permission)) {
    $cls_info = ddu_common::get_class_by_tag($tag);
    if(is_bool($cls)) {
        exit(json_encode(['data' => null, 'msg' => 'error request', 'status' => '-2']));
    }
    $permission_param = $tag_permission[$tag];
    $permit_exist = [];
    foreach($_GET as $param_key => $param) {
        if(in_array($param_key, $permission_param)) {
            $permit_exist[$param_key] = $param;
        }
    }
//    if(count($permit_exist) !== count($permission_param)) {
//        exit(json_encode(['data' => null, 'msg' => 'error request', 'status' => '-3']));
//    }
    $pagesize = 9;
    isset($_GET['pagesize']) && intval($_GET['pagesize']) > 0 && $pagesize = intval($_GET['pagesize']);
    $data = [
        'limit' => ddu_common::get_limit($page+1, $pagesize),
    ];
    $data = array_merge($permit_exist, $data);
    exit(json_encode(['data' => array_values($cls_info['cls']->$cls_info['method']($data)), 'msg' => 'success', 'status' => 0]));
} else {
    exit(json_encode(['data' => null, 'msg' => 'error request', 'status' => '-1']));
}


/**
 * @param $tag
 */
class ddu_common
{
    /**
     * @param $tag
     * @return array|bool
     */
    public static function get_class_by_tag($tag)
    {
        $tag_split_arr = explode('_', $tag);
        if (count($tag_split_arr) < 2) {
            return false;
        }
        $op = $tag_split_arr[0];
        $method = $tag_split_arr[1];
        if (file_exists(PC_PATH . DIRECTORY_SEPARATOR . 'modules' . DIRECTORY_SEPARATOR . $op . DIRECTORY_SEPARATOR . 'classes' . DIRECTORY_SEPARATOR . $op . '_tag.class.php')) {
            return ['cls' => pc_base::load_app_class($op.'_tag', $op), 'method' => $method];
        } else {
            return false;
        }
    }

    /**
     * @param int $pagesize
     * @param int $page
     * @return string
     */
    public static function get_limit( $page = 1, $pagesize = 3)
    {
        return ($page-1) * $pagesize . ',' . $pagesize;
    }
}
```
