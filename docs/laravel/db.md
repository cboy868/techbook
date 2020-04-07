#DB


## 跨库事务

```
DB::beginTransaction();
DB::connection('mysql_third')->beginTransaction();
try {
    $member = Member::find(6);
    $member->race = 't';
    $member->save();

    $mno = MnoInfoThird::find(25);
    $mno->mobile="145888999222";
    $mno->save();

    $member = Member::find(7);
    $member->id = 'abcdd';//这个是错的
    $member->save();

    DB::commit();
    DB::connection('mysql_third')->commit();

} catch (\Exception $e) {
    DB::rollback();
    DB::connection('mysql_third')->rollback();
    Log::error(__METHOD__, [
        'msg' => $e->getMessage()
    ]);
}
```

## 软删除

```
 //软删除使用方法
 use SoftDeletes();
 
 $model = TestSoftDeleteModel::find(1);
 
 // 1、软删除 
 $model->delete();
 
 // 2、恢复数据
 $model->restore()
 
 // 3、强制删除，删除后，数据无法恢复
 $model->forceDelete();
 
 // 4、查找显示包括软删除之内的所有数据
 TestSoftDeleteModel::where('id', '<', 10)->withTrashed()->all();
 
 // 5、只显示所有删除数据
 TestSoftDeleteModel::where('id', '<', 10)->onlyTrashed()->get();

```