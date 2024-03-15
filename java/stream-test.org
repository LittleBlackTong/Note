# stream 练习

### 排序 去重 过滤 限制取值

1. 排序练习

```
	// 对象流练习
	Post post1 = new Post(1, "测试内容", 1500000000001L, (byte) 0, (byte) 0);
	Post post2 = new Post(2, "测试内容", 1500000000002L, (byte) 0, (byte) 0);
	Post post3 = new Post(3, "测试内容", 1500000000003L, (byte) 0, (byte) 1);
	Post post4 = new Post(4, "测试内容", 1500000000004L, (byte) 0, (byte) 0);
	Post post5 = new Post(5, "测试内容", 1500000000005L, (byte) 1, (byte) 1);
	Post post6 = new Post(6, "测试内容5", 1500000000006L, (byte) 0, (byte) 0);
	Post post7 = new Post(7, "测试内容", 1500000000007L, (byte) 0, (byte) 0);

	List<Post> posts = Arrays.asList(post1, post2, post3, post4, post5, post6, post7);
	
	//通过指定的字段进行排序 倒序或者正序
	
	// 都是倒序
	posts.sort(
		Comparator.comparing(Post::getId)
			.thenComparing(Post::getTopPost)
			.reversed()
	);
	
	// 都是正序
	posts.sort(
		Comparator.comparing(Post::getId)
			.thenComparing(Post::getTopPost)
	);
	
	// 先倒序后正序
	posts.sort(
		Comparator.comparing(Post::getId)
			.reversed()
			.thenComparing(Post::getTopPost)
	);
	
	// 先正序后倒序
	posts.sort(
		Comparator.comparing(Post::getId)
			.reversed()
			.thenComparing(Post::getTopPost)
			.reversed()
	);
	
	
	posts.forEach(System.out::println);
	//最后一种描述可能不好理解 先正序然后在倒序 可以理解为 先将 第一部分内容倒序,然后在将所有的结果都倒序排列,那么第一部分的内容就得到了 负负得正的感觉变为了正序
	
```

2. 自定义排序

```
	posts.sort(((o1, o2) -> {
		int result = o1.getId() - o2.getId();
		if (result == 0) {
			result = o1.getTopPost() - o2.getTopPost();
		}
		return result;
	}));
```

3. 去重,取前几位的值,跳过几位在取值

```
	posts = posts.stream()
		.distinct()
		.limit(2)
		.skip(1)
		.collect(Collectors.toList());
	posts.forEach(System.out::println);
```

4. 过滤,匹配

```
	// anyMatch test
	//posts 集合中有没有非置顶的类型
	boolean result = posts.stream()
		.anyMatch(e -> e.getTopPost() == 0);

        //谓词方式
	boolean result2 = posts.stream()
		.anyMatch(Post.idGreaterThanFive);
		
	boolean allMatchResult = posts.stream()
		.allMatch(Post.contentContainsFive);

	boolean isExist = posts.stream()
		.filter(e -> e.getId() > 5)
		.findFirst()
		.isPresent();
```

5. 谓词逻辑

```
	private Integer id;
    private String content;
    private Long createTs;
    private byte isDeleted;
    private byte topPost;

	//声明谓词逻辑 可以进行复用
	public static Predicate<Post> idGreaterThanFive = x -> x.getId() > 5;
    public static Predicate<Post> contentContainsFive = x -> x.getContent().contains("5");
```
