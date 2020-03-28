# 跨库事务

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