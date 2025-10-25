和方块一样，物品是Minecraft中关键的组件。方块组成了你周围的世界，而物品则存在于背包当中。

## 到底什么是物品？
在我们深入到创建物品之前，理解一个物品到底是什么，以及它如何和一个方块区分开来，很重要。让我们用一个例子阐释它：

* 在世界中，你遇到了一个泥土块而且想要采掘它。这是一个**方块**，因为它被放置在了世界中（实际上，它并非一个方块，而是一个方块状态。详见文档的方块状态部分）。
	* 注：不是所有方块都会在被破坏时掉落它们自身（如树叶），详见文档的战利品表部分。
* 当你采掘了该方块后，它就被移除了（=被替换成了空气方块），掉落了泥土。掉落的泥土是一个物品**实体**。这意味着和其他实体一样（猪、僵尸、箭矢等），它能固有地被某些东西移动如水的推动，或被火焰和岩浆烧毁。
* 当你捡起该物品实体后，它变成了一个你背包里的**物品堆叠（item stack）**，一个物品堆叠，简单来说就是一个持有一些额外信息的物品实例，如堆叠数量。
* 物品堆叠（译注：的逻辑）由它对应的**物品**（就是我们要创建的东西）所支持。物品持有含有所有该物品的物品堆叠被初始化时得到的信息的数据组件（例如，每把铁剑都有一个值为250的最大耐久值），同时物品堆叠可以修改那些数据组件，允许同一个物品的不同物品堆叠持有不同的信息（例如，一把铁剑还剩下100点耐久，同时另一把铁剑还剩下200点耐久）。关于什么是通过物品实现的、什么又是通过物品堆叠实现的更多信息，继续往下读。
	* 物品和物品堆叠的关系与方块和方块状态的关系大致相同，在后者中一个方块状态（译注：的逻辑）总是被一个方块支持着。这不是一个非常准确的比较（例如物品堆叠并非单例），但它提供了一个不错的、大致的观点来解释这里的概念。

## 创建一个物品
现在我们理解了什么是一个物品，让我们创建一个吧！

和基本的方块一样，对于不需要特殊功能的基本物品（想想木棍、糖等等），`Item`类可以被直接使用。为了做到这一点，在注册时，用一个`Item.Properties`参数实例化`Item`。这个`Item.Properties`参数可以由`Item.Properties#of`（译注：原文如此，实际上应该是直接new `Item.Properties`）创建，以及可以通过调用如下方法来自定义内容：

* `stacksTo` - 设置该物品的最大堆叠数量（通过`DataComponents#MAX_STACK_SIZE`）。默认为64。例如，该方法被用于末影珍珠以及其他只能最大堆叠至16的物品。
* `durability` - 设置该物品的最大耐久值（通过`DataComponents#MAX_DAMAGE`），并设置初始损害值（通过`DataComponents#DAMAGE`）为0。默认为0，意为“没有耐久值”。例如，铁制工具在这里使用值250。注：设置最大耐久值会自动将最大堆叠数量锁死为1。
* `craftRemainder` - 设置该物品的合成剩余物。原版给装满了的桶使用了这个，会在合成后留下空桶。
* `fireResistant` - 让该物品对应的物品实体免疫火焰和岩浆（通过`DataComponents#FIRE_RESISTANT`）。被各种下界合金物品所使用。
* `setNoRepair` - 禁用铁砧和合成格修复该物品。未被原版使用。
* `rarity` - 设置该物品的稀有度（通过`DataComponents#RARITY`）。目前，它只影响物品的（译注：文本）颜色。`Rarity`是一个包含四个值的枚举类型：`COMMON`（白色，默认值）、`UNCOMMON`（黄色）、`RARE`（浅绿色）和`EPIC`（亮紫色）。注意Mod也许会添加更多稀有度类型。
* `requiredFeatures` - 设置该物品所需的特性标志。它主要用于原版的小版本的特性锁机制。不建议使用这个，除非你在与一个被锁在特性标志之后的系统进行交互。
* `food` - 设置该物品的`FoodProperties`（通过`DataComponents#FOOD`）。

为举例子，或者看看被Minecraft使用的各种值，看看`Items`类。

### 食物
`Item`类给食物物品提供了默认的功能，意味着你不需要为此创建一个类。为了让你的物品可食用，你只需要在`Item.Properties`中通过调用`food`方法来设定`FoodProperties`。

`FoodProperties`由`FoodProperties.Builder`创建。你可以给它设置各种属性：

* `nutrition` - 设定恢复多少饥饿度。以半个饥饿度（译注：半个饥饿度图标）为单位计数，例如，Minecraft原版的牛排恢复8点饥饿度。
* `saturationMod` - 用于计算恢复的饱食度的饱食度修饰符。计算公式为`min(2 * nutrition * saturationMod, playerNutrition)`，意味着使用0.5会让恢复的饥饿度和饱食度相同。
* `alwaysEdible` - 该物品是否总是能被食用，即使饥饿条是满的。默认为`false`，金苹果等不只是填充饥饿条而还提供额外加成的物品使用`true`。
* `fast` - 快速食用是否应该被启用。默认为`false`，原版的干海带使用`true`。
* `effect` - 添加一个食用时给予的`MobEffectInstance`。第二个参数标志着该`MobEffectInstance`被应用的可能性；例如，腐肉有着80%（=0.8）的可能性在食用时应用饥饿效果。这个方法有两个变体；你应该用那个参数为一个supplier的变体（另一个参数直接为一个`MobEffectInstance`，由于类加载问题，被NeoForge弃用了）。
* `usingConvertsTo` - 设置该物品在被使用后会变成什么。
* `build` - 当你设置好一切你想要设置的东西，调用`build`来得到一个`FoodProperties`以作他用。

为举例子，或者看看被Minecraft使用的各种值，看看`Foods`类。

为了得到某个物品的`FoodProperties`，调用`Item#getFoodProperties(ItemStack, LivingEntity)`。这可能会返回null，毕竟不是每个物品都可食用。为了区分某个物品是否可食用，对`getFoodProperties`的返回结果判空。

### 更多功能
直接使用`Item`类只适用于非常基础的物品。如果你想要添加更多功能，例如右键交互，需要一个自定义的继承`Item`的类。`Item`类有很多方法可以覆写去做很多不同的事；详见`Item`和`IItemExtension`。

两个最常见的物品用例是左击和右击。对于左击，请见文档的破坏一个方块和击打一个实体（未完成）部分。对于右击，详见文档的交互管线部分。

### DeferredRegister.Items
所有注册表使用`DeferredRegister`来注册它们的内容，物品也不例外。然而，鉴于添加新物品是非常之多的Mod的必要特性，NeoForge提供了`DeferredRegister.Items`这一帮助类，其继承了`DeferredRegister<Item>`，提供了一些物品特有的辅助方法：

```Java
public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(ExampleMod.MOD_ID);

public static final Supplier<Item> EXAMPLE_ITEM = ITEMS.registerItem(
"example_item",
Item::new, //属性参数将被传入的工厂方法。
new Item.Properties() //使用的属性参数。
);
```

在内部实现中，它会通过将属性参数传入提供的物品工厂方法（一般是构造器），调用`ITEMS.register("example_item", () -> new Item(new Item.Properties()))`。

如果你想使用`Item::new`，你可以不填工厂方法，使用方法的`simple`变体：

```Java
public static final Supplier<Item> EXAMPLE_ITEM = ITEMS.registerSimpleItem(
"example_item",
new Item.Properties() //使用的属性参数。
);
```

它起到的效果和之前的例子是一样的，但略微短了一点。当然，如果你想使用一个`Item`子类而非`Item`本身，你就需要使用之前的方法了。

这两个方法都有省略`new Item.Properties()`参数的重载：

```Java
public static final Supplier<Item> EXAMPLE_ITEM = ITEMS.registerItem("example_item", Item::new);
//也省略了Item::new参数的变体方法。
public static final Supplier<Item> EXAMPLE_ITEM = ITEMS.registerSimpleItem("example_item");
```

最后，还有为方块物品准备的shortcut：

```Java
public static final Supplier<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
"example_block",
ExampleBlocksClass.EXAMPLE_BLOCK, new Item.Properties()
);
//省略属性参数的变体：
public static final Supplier<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
"example_block",
ExampleBlocksClass.EXAMPLE_BLOCK
);
//省略名称参数的变体，使用对应方块的注册名：
public static final Supplier<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
ExampleBlocksClass.EXAMPLE_BLOCK,
new Item.Properties()
);
//名称参数和属性参数均省略的变体：
public static final Supplier<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
ExampleBlocksClass.EXAMPLE_BLOCK
);
```

> 注：如果你将注册了的方块放在一个分隔开的类里，你需要在你的物品类之前加载你的方块类。

### 资源
如果你注册了你的物品并获取了它（通过`/give`或创造模式物品栏），你会发现它缺少一个正确的模型和纹理。这是因为纹理和模型是由Minecraft的资源系统处理的。

要将一张简单的纹理应用于一个物品，你必须添加一个物品模型JSON和一张纹理PNG。详见文档的资源部分的对应部分。

## ItemStack
像方块和方块状态一样，大多数你需要一个`Item`的情况传入的实际上是一个`ItemStack`。物品堆叠（`ItemStack`）代表了一个或多个物品在一个容器（例如背包）中的堆叠。也同样和方块和方块状态一样，应该覆写`Item`的方法，调用`ItemStack`的方法，许多`Item`中的方法也有一个`ItemStack`参数传进来。

一个`ItemStack`包括了三个主要的部分：

* 它代表的`Item`（译注：物品类型），可通过`ItemStack#getItem`得到。
* 堆叠数量，通常在1到64之间，可通过`getCount`得到，可通过`setCount`或`shrink`改变。
* 数据组件字典（data component map），存储着每个物品堆叠特定的数据。可通过`getComponents`得到。组件的值通常由`has`、`get`、`set`、`update`和`remove`来得到或更改。

要创建一个`ItemStack`，调用`new ItemStack(Item)`，传入对应的物品。默认情况下，它会使用1的堆叠数量，没有NBT数据（译注：1.20.5及以后`ItemStack`不存在NBT数据了，这里可能是忘记改了）；也有接受堆叠数量和NBT数据的重载，如果需要的话。

`ItemStack`是可变对象（见下文），然而有些时候需要将它们作为不可变对象处理。如果你需要更改一个被视作不可变对象的`ItemStack`，你可以通过`#copy`来复制它，或`copyWithCount`，如果一个特定的堆叠数量应该被使用的话。

如果你想要表示一个没有物品的物品堆叠，用`ItemStack.EMPTY`。如果你想要知道一个`ItemStack`是否为空，调用`#isEmpty`。

### ItemStack的可变性
`ItemStack`是可变的对象。这意味着如果你调用例如`#setCount`或任何和数据组件字典相关的方法，`ItemStack`自身会被修改。原版广泛地利用`ItemStack`的可变性，数个方法也依赖于此。例如，`#split`方法分割它调用的`ItemStack`的数量，在处理过程中同时修改原`ItemStack`、返回一个新的`ItemStack`。

然而，在处理多个`ItemStack`时这可能导致问题。最常见的例子是当处理背包物品栏的时候，因为你需要同时考虑当前被光标选中的`ItemStack`，以及你尝试去放入/取出的`ItemStack`。

> 注：当不确定时，最好有备无患，调用`#copy`复制该物品堆叠。

### JSON表达形式
在很多情况下，例如配方中，物品堆叠需要以JSON对象的形式表达。一个物品堆叠的JSON表达形式看起来就像下面这样：

```JSON
{
	//物品的ID，必需项。
	"id": "minecraft:dirt",
	//物品堆叠数量。可选项，默认为1。
	"count": 4,
	//一个数据组件的字典。可选项，默认为一个空字典。
	"components": {
		"minecraft:enchantment_glint_override": true
	}
}
```

## 创造模式物品栏
默认情况下，你的物品只能通过`/give`获得而且不会出现在创造模式物品栏中。让我们改变这个情况！

你将你的物品加入到创造模式菜单的方法取决于你想将它加入到哪个创造模式物品栏中。

### 已有的创造模式物品栏
> 注：这个方法用于将你的物品加入到Minecraft或其他Mod的创造模式物品栏中。要加入到你自己的创造模式物品栏，看下文。

一个物品可以通过`BuildCreativeModeTabContentsEvent`被加入到一个已有的`CreativeModeTab`，该事件只在Mod事件总线、逻辑客户端上运行。调用`event#accept`来添加新物品。

```Java
//MyItemsClass.MY_ITEM是一个Supplier<? extends Item>, MyBlocksClass.MY_BLOCK是一个Supplier<? extends Block>
@SubscribeEvent //在Mod事件总线上
public static void buildContents(BuildCreativeModeTabContentsEvent event) {
	//这个是我们要添加物品的创造模式物品栏吗？
	if (event.getTabKey() == CreativeModeTabs.INGREDIENTS) {
		event.accept(MyItemsClass.MY_ITEM.get());
		//接受一个ItemLike。这假定了MY_BLOCK有一个相应的物品。
		event.accept(MyBlocksClass.MY_BLOCK.get());
	}
}
```

这个事件还提供了一些额外信息，如`getFlags`来获取被允许的特性标志的列表，或`getPermissions`来检查该玩家是否有权限查看操作的创造模式物品栏。

### 自定义创造模式物品栏
`CreativeModeTab`有一个注册表，意味着自定义的`CreativeModeTab`必须被注册。创建一个创造模式物品栏使用一个创建者（builder）系统，builder可通过`CreativeModeTab#builder`获得。builder提供了设置标题、图标、默认的物品以及一系列其他属性的选项。另外，NeoForge提供了额外的方法来自定义创造模式物品栏的图片、标签、槽的颜色、该物品栏应该被排序到哪里，等等。

```Java
//CREATIVE_MODE_TABS是一个DeferredRegister<CreativeModeTab>
public static final Supplier<CreativeModeTab> EXAMPLE_TAB = CREATIVE_MODE_TABS.register("example", () -> CreativeModeTab.builder()
	//设置该创造模式物品栏的标题。不要忘了添加翻译！
	.title(Component.translatable("itemGroup." + MOD_ID + ".example"))
	//设置该创造模式物品栏的图标。
	.icon(() -> new ItemStack(MyItemsClass.EXAMPLE_ITEM.get()))
	//将你的物品加入该创造模式物品栏。
	.displayItems((params, output) -> {
		output.accept(MyItemsClass.MY_ITEM.get());
		//接受一个ItemLike。这假定了MY_BLOCK有一个相应的物品。
		output.accept(MyBlocksClass.MY_BLOCK.get());
	})
	.build()
);
```

## ItemLike
`ItemLike`是一个在原版被`Item`和`Block`（译注：及其子类）实现了的接口。它定义了方法`asItem`，无论对象到底是什么，返回它代表的物品：`Item`只是返回其自身，`Block`返回它们的相关联的`BlockItem`，如果存在的话，否则返回`Blocks.AIR`（译注：这里应该是`Items.AIR`）。`ItemLike`被用于很多物品的“来源”不重要时的上下文中，例如在很多数据生成器中。

为你的自定义对象实现`ItemLike`也是可能的。只需要覆写`asItem`即可。
