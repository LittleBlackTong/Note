# stream map 练习

> 和 clojure 里面的 map 理解 是一样的,可以分别获取流中的每一个元素,然后在对其做一些操作

### map 基础练习

```
1. 遍历字符串集合然后转换成大写字母

public static void main(String[] args) {
	//之前的方式
	List<String> names = Arrays.asList("crazy", "nana", "mangmang", "coco", "leron");
	List<String> newNames = new ArrayList<>();
	for (String name : names) {
		newNames.add(name.toUpperCase());
	}
	System.out.println(newNames);

	// stream map 方式
	newNames = names.stream().map(String::toUpperCase).collect(Collectors.toList());
	System.out.println(newNames);
}
```

### map 处理对象集合

```
public static void main(String[] args) {
        // 对象流练习
        Post post1 = new Post(1, "测试内容", 1500000000001L, (byte) 0, (byte) 0);
        Post post2 = new Post(2, "测试内容", 1500000000002L, (byte) 0, (byte) 0);
        Post post3 = new Post(3, "测试内容", 1500000000003L, (byte) 0, (byte) 1);
        Post post4 = new Post(4, "测试内容", 1500000000004L, (byte) 0, (byte) 0);
        Post post5 = new Post(5, "测试内容", 1500000000005L, (byte) 1, (byte) 1);
        Post post6 = new Post(6, "测试内容5", 1500000000006L, (byte) 0, (byte) 0);
        Post post7 = new Post(7, "测试内容", 1500000000007L, (byte) 0, (byte) 0);

        List<Post> posts = Arrays.asList(post1, post2, post3, post4, post5, post6, post7);
		// peek 相当于 map 的另一种实现 可以不用在接口中添加返回值 return e;
		posts = posts.stream()
                .peek(
                        e -> {
                            e.setContent("hello world");
                            e.setTopPost(e.getTopPost() == 1 ? (byte) 0 : (byte) 1);
                        }
                ).collect(Collectors.toList());
        System.out.println(posts);
		
		//map 版
			posts = posts.stream()
                .map(
                        e -> {
                            e.setContent("hello world");
                            e.setTopPost(e.getTopPost() == 1 ? (byte) 0 : (byte) 1);
							return e;
                        }
                ).collect(Collectors.toList());
        System.out.println(posts);
		
}
```

### flatMap 处理 stream 嵌套 和 集合嵌套时 使用的 map 方法

1. 实例

```
	List<String> words = Arrays.asList("hello", "world");
	words.stream()
		.flatMap(w -> Arrays.stream(w.split("")))
		.forEach(System.out::println);
```
