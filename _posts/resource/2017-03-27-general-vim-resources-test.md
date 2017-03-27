---
layout: post
title: Vim 测试代码
category: 资源
tags: Vim
keywords: Vim
---

## 在粘贴代码时不启动自动缩进

粘贴之前输入 `:set paste`
粘贴完后恢复 `:set nopaste`

## 关闭和开启行号

关闭 `:set nonu`
开启 `:set number`
代码 
```
//将名称换成半角
    public function chenge_string()
    {
        $meituan = DB::connection('mysql_shpinx')->table('baidu_tenantinfo')
            ->select('id', 'name')
            ->where('name', 'like', '%' . '（' . '%' . '）' . '%')
            ->orderBy('id')
            ->get();
//        dd($meituan);
//        set_time_limit(300);
        foreach ($meituan as $key => $val) {
            $new_name = CommonMethod::convertStrType($val->name);
            $val->name = $new_name;
            $result = DB::connection('mysql_shpinx')->table('baidu_tenantinfo')
                ->where('id', $val->id)
                ->update(['name' => $new_name]);
            echo($val->id . '_____' . $result);
            echo '<br>';
        }
    }
```

