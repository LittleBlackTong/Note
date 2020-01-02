# NewsFeed 后端存在问题总结 以及优化建议

-------------------

### 一、帖文 和 投票 结构问题
```
1.获取`全部帖文`以及`个人主页`和`群组帖文`查询逻辑

public static final String HOMEPAGE_POST_SQL = "select * from post where id in \n" +
            "(\n" +
            "select a.id from (\n" +
            "select id, type from \n" +
            "(\n" +
            "select id, create_ts, \"post\" as type from post where ((user_id = ?1 and type = 1) " +
            "or (user_id in (select follow_user_id from follow where user_id = ?1) and type = 1) " +
            "or (group_id in (select b.id from group_members a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0) and type = 0)" +
            "or (group_id in (select b.id from group_follow  a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0) and type = 0 and is_public = 1))" +
            "and state = 1 and ifnull(content, \"\") like ?2 \n" +
            "union all\n" +
            "select id, create_ts, \"vote\" as type from vote where ((group_id in (select b.id from group_members a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0)) " +
            "or (group_id in (select b.id from group_follow a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0))) " +
            "and status = 1 and ifnull(content, \"\") like ?2 \n" +
            ") as homePage where (create_ts between ?3 and ?4) order by create_ts desc limit ?5,?6\n" +
            ") as a where type = \"post\"\n" +
            ")";

public static final String HOMEPAGE_VOTE_SQL = "select * from vote where id in \n" +
            "(\n" +
            "select a.id from (\n" +
            "select id, type from \n" +
            "(\n" +
            "select id, create_ts, \"post\" as type from post where ((user_id = ?1 and type = 1) " +
            "or (user_id in (select follow_user_id from follow where user_id = ?1) and type = 1) " +
            "or (group_id in (select b.id from group_members a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0) and type = 0)" +
            "or (group_id in (select b.id from group_follow  a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0) and type = 0 and is_public = 1))" +
            "and state = 1 and ifnull(content, \"\") like ?2 \n" +
            "union all\n" +
            "select id, create_ts, \"vote\" as type from vote where ((group_id in (select b.id from group_members a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0)) " +
            "or (group_id in (select b.id from group_follow a, groups b where a.group_id = b.id and a.user_id = ?1 and b.is_deleted = 0 and b.is_closed = 0))) " +
            "and status = 1 and ifnull(content, \"\") like ?2 \n" +
            ") as homePage where (create_ts between ?3 and ?4) order by create_ts desc limit ?5,?6\n" +
            ") as a where type = \"vote\"\n" +
            ")";
```
> 实现逻辑：将帖文和投票 通过 union all 的方式将 投票表和帖文表组合在一起然后进行分页查询
> 缺点 加载单独帖文页面时 需要帖文和投票需要结合业务逻辑分别查询 然后将查询的帖文和投票组合在一起在程序里进行排序返回，其中还包含这其他数据的包装，导致帖文加载数据慢，个人理解这个不利于扩展，如果新增帖文类型相当于重构 newsfeed
> 修改想法：更改数据结构，将帖文和投票融合成一张表，投票变为新表的一个类型属性，帖文是核心这个修改大工程

### 二、点赞数量，分享数量问题
```
1.帖文加载时需要包装点赞数量 目前方式是每条帖文都进行此操作

select count（id） from likes where ....
```
> 点赞这个地方涉及到一个并发的问题，多人同时对一个帖文进行点赞，如果在数据库里面单独维护一个点赞数量的字段，并发进行时会导致点赞数量减少的问题。
> 将帖文的点赞数量维护在缓存中，在加载帖文时，点赞数量由缓存来获取提高帖文的加载速度。

### 三、用户一致性问题
```
1.newsfeed 加载群组成员数据 或者 加载用户身份数据原来的方式
Object user = redisTemplate.opsForHash().get(RedisKey.NEWSFEED_USERS_KEY, userId);

 Map<Object, Object> allUser = redisTemplate.opsForHash().entries(RedisKey.NEWSFEED_USERS_KEY);
        List<Integer> userIds = new ArrayList<>();
        for (String searchName : searchNames) {
            for (Object key : allUser.keySet()) {
                JSONObject jsonObject = JSONObject.fromObject(allUser.get(key));
                String name = jsonObject.getString("name");
                if (name.contains(searchName)) {
                    userIds.add(Integer.valueOf(key.toString()));
                }
            }
   		}
```
> 原来 newsfeed 在加载和用户相关的数据时通过 redis 获取这个需要一个小时同步一次导致数据不一致，还有搜索群组成员时，由于newsfeed 本身没有用户属性的数据需要到缓存里遍历 10000个用户数据得到结果然后在 组成 list 返回给前端，导致分页时必须查询遍历全部用户才可以加载出来，个人认为效率很低，即使是在缓存里，每次查询就需要遍历一次 10000个用户缓存也不太好....

>修改想法：newsfeed 不在保存任何和用户相关的内容，但是会保留用户和群组的关联关系，需要查询成员相关内容时直接去 usercenter 获取根据逻辑进行分页处理实现正确的服务拆分。


### 四、各种状态问题 如邀请用户，申请入群
>目前这个状态的判断是和通知数据进行判断的，当用户拥有申请入群的通知，那么认为这个用户状态是申请中的，目前看虽然想要的功能能实现，但是感觉很恶心。用户的各种状态应该单独拿出来变为一张表，而不是通过通知来判断的。

### 五、分享帖文套娃问题
> 分享帖文后端逻辑是将被分享的帖文包装到新的帖文中进行分享，如果帖文在分享的基础上继续分享，会认为分享的帖文是一个新的帖文，那么就会需要递归逻辑遍历获取最原始的帖文
>缺点：当分享次数过多程序加载速度会出现问题，递归本身这种逻辑就在这个场景就不太好
>修改想法： 将帖文分享，分享的帖文永远为初始帖文这样永远只要去被分享帖文内容就可以了。提升运行速度。

### 六、评论的表结构问
> 评论表结构目前为了实现需求，在开发过程中添加了很多重复字段，可以重新设计对应的表结构达到更简洁的目的

### 七、NewsFeed 服务的想法
如图：

![newsfeed-frameword](https://github.com/LittleBlackMann/Note/blob/master/Image/newsfeed-future.jpg?raw=true "newsfeed-framework")
