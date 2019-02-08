# 运用您的 Rust 技能构建一个简单的井字棋游戏
- https://www.ibm.com/developerworks/cn/opensource/os-using-rust/index.html?ca=drs-
- Dylan Hicks

我非常喜欢 Rust。这种静态编译语言是内存安全且与操作系统无关的，所以它可以在任何计算机上运行。Rust 带来了系统语言的速度和低阶优势，而没有 C# 和 Java 等语言中麻烦的垃圾收集。

要学习一种语言，最好的方法就是开始实际使用它。本文通过展示如何使用 Rust 构建一个简单的井字棋游戏，帮助您运用该语言。跟随我构建您自己的有趣游戏吧。

## 启动项目

首先，您需要设置您的项目。可以使用 Cargo 从终端创建新的二进制可执行程序：

```
$ cd ~/Documents
$ cargo new tic_tac_toe –bin
```

在 tree 程序中，新的 tic_tac_toe 目录如下所示：

```
$ cd tic_tac_toe
$ tree .
.
??? Cargo.toml
??? src
    ??? main.rs
```

main.rs 文件应包含以下几行内容：

```
fn main() {
    println!("Hello, world!");
}
```

运行该程序就像创建它一样简单，如 清单 1 所示。
清单 1. 运行“Hello, World!”
```
$ cargo build

    Compiling …

     Finished …

$ cargo run

     Finished …

      Running …

Hello, world!
```

现在，您还需要为游戏模块创建一个文件。执行以下命令行来创建此文件：

创建了项目和目录后，就可以深入研究游戏的轮廓了。
## 规划游戏的类型和结构

经典的井字棋游戏由两个主要组件组成：一个棋盘和每位玩家的轮次。棋盘实质上是一个空的 3x3 数组，轮次表明哪位玩家必须落子。要将此功能转变为代码，必须编辑上一节中创建的 game.rs 文件（参见清单 2）。
清单 2. 针对棋盘和玩家轮次进行了修改的 game.rs
```
type Board = Vec<Vec<String>>;

enum Turn {

    Player,

    Bot,

}

pub struct Game {

    board: Board,

    current_turn: Turn,

}
```

您可能已经注意到这里的奇怪语法，但不必担心：我会逐步介绍它。
### 棋盘

要将游戏棋盘转变为代码，可以使用 type 关键字设置名称 Board 的别名，以便与类型 Vec<Vec<String>> 同义。现在，Board 是一个二维字符串矢量的简单类型。我会在这里使用一个字符，因为该数组中的值只有 x、o 或表示一个开放位置的数字。
### 轮次

轮次表示哪位玩家必须选择一个位置，所以 enum 结构非常适合。在每轮中，只需匹配 Turn 变量来执行合适的方法调用。
### 游戏

最后，您必须创建一个包含棋盘和当前轮次的 Game 对象。但是等等！Game 结构的方法在哪里？别担心：接下来就会介绍。
## 实现游戏

井字棋游戏由哪些方法组成？有许多轮次。在每轮中，都会显示棋盘，一个玩家落一颗棋子，再次显示棋盘，然后检查获胜条件。如果游戏分出胜负，该游戏会宣布哪位玩家获胜，并邀请他或她再玩一局。如果没有人在游戏中获胜，该游戏会切换当前玩家并进入游戏的下一轮次。显然，根据具体的玩家，每一次落子都涉及一些细微问题，您可以从这里开始钻研。

首先，创建一个嵌套在 impl 代码块中的构造，如清单 3 所示。
清单 3. game 构造

```
impl Game {

    pub fn new() -> Game {

        let first_row = vec![

            String::from("1"), String::from("2"), 

            String::from("3")];

        let second_row = vec![

            String::from("4"), String::from("5"), 

            String::from("6")];

        let third_row = vec![

            String::from("7"), String::from("8"), 

            String::from("9")];

        Game {

            board: vec![first_row, second_row, third_row],

            current_turn: Turn::Player,

        }

    }

}
```

静态方法 new 创建并返回一个 Game 构造。这是 Rust 中的对象构造函数的标准名称。

您必须将 board 成员变量与一个 String 对象的 2d 矢量绑定在一起。请注意，我在每个位置中都填入了一个数字，表明每次落子的可用位置，而不是将每个位置都留空。接下来，将 current_turn 成员变量绑定到 Turn::Player 的值。这一行表示每局游戏都让玩家先走。
### 您如何玩该游戏？

第一个方法用作程序的映射。您将此方法（连同本节的剩余方法）添加到 impl Grid 代码块内。清单 4 给出了该方法。
清单 4. 游戏程序的映射
```
pub fn play_game(&mut self) {

    let mut finished = false;

    while !finished {

        self.play_turn();

        if self.game_is_won() {

            self.print_board();

            match self.current_turn {

                Turn::Player => println!("You won!"),

                Turn::Bot => println!("You lost!"),

            };

            finished = Self::player_is_finished();

            self.reset();

        }

        self.current_turn = self.get_next_turn();

    }

}
```

很容易看出该游戏的流程。使用一个无限循环，从一轮前进到下一轮，并轮换 current_turn。出于这个原因，您将在 self 上使用了一个可变借用，因为游戏的内部状态会在每轮中发生更改。

这个 enum 已获得了回报，因为如果游戏分出胜负，就会嵌入有关谁赢得游戏的信息。然后让玩家知道他或她是赢了还是输了。此外，您将棋盘重置到原始状态，这在用户想再玩一局时很有帮助。

请注意，这将是 new 以外的唯一一个 pub 方法。这意味着在使用 Game 对象时，play_game 和 new 是另一个库能够访问的唯一方法。其他所有方法都是私有的，无论它们是否是静态方法。
### 互换位置

play_game 方法中使用的第一个帮助器方法是 play_turn。清单 5 给出了这个小巧的函数。
清单 5. play_turn 函数

```
fn play_turn(&mut self) {

    self.print_board();

    let (token, valid_move) = match self.current_turn {

        Turn::Player => (

            String::from("X"), self.get_player_move()),

        Turn::Bot => (

            String::from("O"), self.get_bot_move()),

    };

    let (row, col) = Self::to_board_location(valid_move);

    self.board[row][col] = token;

}
```

这个函数比较复杂。首先，您要输出一个棋盘，让用户知道哪些位置可以落子（甚至在轮到机器人落子时也很有用）。接下来，根据 current_turn 变量，您可以使用元组解构和 match 来分配变量 token 和 valid_move。

对于玩家或机器人，token 分别为 String X 或 O。valid_move 为 1 到 9 的整数，表示棋盘上未占用的位置。然后，使用 to_board_location 静态方法，将此变量转换为棋盘的相应行和列。（带有大写“S”的 Self 返回一个 self 类型 - 在本例中为 Game。）
让我们看看棋盘

设置 play_turn 后，您需要一个输出方法。清单 6 给出了该方法。
清单 6. 输出游戏棋盘

```
fn print_board(&self) {

    let separator = "+---+---+---+";

    println!("\n{}", separator);

    for row in &self.board {

        println!("| {} |\n{}", row.join(" | "), separator);

    }

    print!("\n");

}
```

在此方法中，您使用一个 for 循环来输出棋盘上各行的 ASCII 表示。临时变量 row 是对棋盘上每个矢量的引用。通过使用 join 方法，您可以将 row 转换为 String，并输出新值和一个附加的分隔符 String。

输出功能生效后，最后可以继续为玩家和机器人实现有效落子。
玩家，轮到您了

目前，此程序是一系列硬编码的返回值，没有来自玩家的输入。清单 7 将改变这一状况。
清单 7. 设置轮换

```
fn get_player_move(&self) -> u32 {

    loop {

        let mut player_input = String::new();

        println!(

            "\nPlease enter your move (an integer between \

            1 and 9): ");

        match io::stdin().read_line(&mut player_input) {

            Err(_) => println!(

                "Error reading input, try again!"),

            Ok(_) => match self.validate(&player_input) {

                Err(err) => println!("{}", err),

                Ok(num) => return num,

            },

        }

    }

}
```

此方法的核心可归纳为：除非玩家为游戏提供了一步有效的落子，否则它会无限循环。

用户提示后的第一个 match 表达式尝试将用户的输入读到一个 String (player_input) 中，并检查这样做是否会出错。io 模块提供了这项功能；您必须在 game.rs 文件顶部导入此模块。它的 stdin().read_line 方法 (stdin()) 向当前标准输入返回一个句柄对象。这是我导入 io 模块的语句：

同样值得注意的是，尽管 read_line 方法修改了给定的 String，但它也返回了一个名为 Result 的 enum。我在介绍性文章中没有谈到 Result，所以接下来讲一下它。
### Result enum

Result 被认为是一种代数类型。它是一个 enum，包含两个变量：Ok 和 Err。每个变量都可以包含数据，比如 String 或 i32。

对于 read_line，返回的 Result 是一个来自 io 模块的特殊版本，这意味着 Err 是一个特殊的 io::Error 变量。相较而言，Ok 与原始的 Result 变量相同，而且在这里包含一个表示读取的字节数的整数。Result 是一个有用的 enum，有助于确保您在编译时而不是运行时解决所有可能的错误。

另一个普遍存在于 Rust 中的类似 enum 是 Option。它的变量为 None（不含数据）和 Some（含数据），而不是 Ok 和 Err。Option 很有用，其用途类似于 C++ 中的 nullptr 或 Python 中的 None。

Option 和 Result 之间有何区别，应在何时使用它们？以下是我认可的答案。首先，如果您期望一个函数什么都不返回，那么可以使用 Option。如果您期望函数总是成功但可能失败，这意味着必须捕获错误，那么可以使用 Result。明白了吗？很好。返回到 get_player_move 方法。
### 返回到游戏中

前面讲到从玩家读取输入。如果读取用户输入时发生错误，程序会告知用户并要求他或她重新输入。如果未发生错误，程序会到达第二个 match 表达式。请注意下划线 (_) 的使用：它们告诉 Rust，不同于在第二个 match 表达式中的操作，您没有将数据绑定到 Result 的 Ok 或 Err 变量内。

这个 match 表达式会检查 player_input 变量是否有效。如果该变量是无效的，那么代码会返回一个错误（游戏会提醒玩家注意该错误），并请求玩家提供有效的输入。如果 player_input 有效的，则返回该输入（使用 validate 方法转换为一个整数）。
验证您的代码

编写了游戏的核心代码后，编写一个 validate 函数会很不错。清单 8 给出了相关代码。
清单 8. Validate 函数

```
fn validate(&self, input: &str) -> Result<u32, String> {

    match input.trim().parse::<u32>() {

        Err(_) => Err(

            String::from(

                "Please input a valid unsigned integer!")),

        Ok(number) => {

            if self.is_valid_move(number) {

                Ok(number)

            } else {

                Err(

                    String::from(

                        "Please input a number, between \

                        1 and 9, not already chosen!"))

            }

        }

    }

}
```

通过逐行浏览此输出，可以得出该方法的大体含义如下。

首先，程序返回一个 Result enum。我没有介绍类型模板，但实质上，您指示了 Result 的 Ok 变量必须包含一个 u32 整数，Err 变量必须包含一个 String。为什么这里会返回一个 Result？该方法预计会通过，仅在给定输入满足以下条件时才会抛出错误：

    不是整数；
    由于被占用而不是一个有效的位置；或者
    由于该整数不在 1–9 内而不是一个有效的位置。

接下来，程序尝试使用 input 的 parse 方法将 input 转换为 u32。turbofish, ::<type> 是一些函数的一个特殊方面，它会告诉这些函数返回何种类型。在本例中，它告诉 parse 尝试将 input 转换为 u32，同时设置 Result 的 Ok 变量来包含一个 u32。如果 input 无法转换，代码会返回一个错误，表明 input 不是一个无符号整数。但是，如果成功转换，代码会将 input 传递给另一个帮助器函数：is_valid_move。

为什么有另一个用于验证的帮助器函数？在之前的可能错误列表中，第一个是特定于用户的。机器人始终会提供一个整数。因此，您只需使用 validate 来验证玩家的响应。is_valid_move 检查另外两个可能的错误。

清单 9 给出了验证代码的最后部分。
清单 9. 更多验证

```
fn is_valid_move(&self, unchecked_move: u32) -> bool {

    match unchecked_move {

        1...9 => {

            let (row, col) = Self::to_board_location(

                unchecked_move);

            match self.board[row][col].as_str() {

                "X" | "O" => false,

                 _ => true,

            }

        }

        _ => false,

    }

}
```

非常简单。如果给定的 unchecked_move 不在 1 和 9（含 1 和 9）之间，那么它不是一次有效的落子。否则，代码必须检查该位置是否已落子。像之前在 play_turn 中一样，您将 unchecked_move 转换为棋盘上的相应行和列。然后可以检查该位置是否在棋盘上。如果该位置为 X 或 O，则落子是无效的。
### 轮到机器人了

在编写方法来实现机器人的落子之前，请创建清单 10 中所示的 to_board_location 静态方法。
清单 10. to_board_location 方法

```
fn to_board_location(game_move: u32) -> (usize, usize) {

    let row = (game_move - 1) / 3;

    let col = (game_move - 1) % 3;

    (row as usize, col as usize)

}
```

此方法稍微有点欺骗性，因为众所周知，在 validate 和 play_turn 中调用 to_board_location 时，参数 game_move 是介于 1 和 9（含 1 和 9）之间的整数。您将此方法设置为静态方法，因为此数学运算与 Game 对象毫无关系。井字棋的棋盘始终为 3x3。
### 对弈机器人

您的代码可以实现玩家的落子，但是如何实现机器人的落子呢？首先，机器人的落子应该是一个随机数，这意味着您需要导入第三方 crate rand。其次，您不断生成随机落子，直到使用 is_valid_move 方法确定它到达一个有效位置。然后，该游戏必须向玩家告知机器人的落子，并返回该落子的信息。

您导入该 rand crate 并安装在一个名为 Cargo.toml 的文件中，使用 rand 作为依赖项。清单 11 给出了该文件。
清单 11. Cargo.toml
```
[package]

name = "tic_tac_toe"

version = "0.1.0"

authors = ["Dylan Hicks <dirtgrub.dylanhicks@gmail.com>"]

[dependencies]

rand = "0.4"
```

main.js 文件告诉 Cargo，您想使用此依赖项。我将此命令放在文件顶部：

然后，将此命令放在 game.rs 文件的顶部，放在 io 导入语句上方：

有了 rand crate 来生成随机数字后，您需要一个方法来实现机器人的落子。清单 12 给出了该方法。
清单 12. bot_move 方法

```
fn get_bot_move(&self) -> u32 {

    let mut bot_move: u32 = rand::random::<u32>() % 9 + 1;

    while !self.is_valid_move(bot_move) {

        bot_move = rand::random::<u32>() % 9 + 1;

    }

    println!("Bot played moved at: {}", bot_move);

    bot_move

}
```

这很简单，对吧？

该方法使用了 play_turn 方法的依赖项。 现在，您需要创建一个方法来检查游戏是否分出胜负。
我们是冠军

现在，您将快速轻松地执行一些布尔代数运算 (清单 13)。
清单 13. 一些布尔代数运算
```
fn game_is_won(&self) -> bool {

    let mut all_same_row = false;

    let mut all_same_col = false;

    for index in 0..3 {

        all_same_row |= 

            self.board[index][0] == self.board[index][1]

            && self.board[index][1] == self.board[index][2];

        all_same_col |= 

            self.board[0][index] == self.board[1][index]

            && self.board[1][index] == self.board[2][index];

    }

    let all_same_diag_1 =

        self.board[0][0] == self.board[1][1] 

        && self.board[1][1] == self.board[2][2];

    let all_same_diag_2 =

        self.board[0][2] == self.board[1][1] 

        && self.board[1][1] == self.board[2][0];

        (all_same_row || all_same_col || all_same_diag_1 || 

         all_same_diag_2)

}
```

在 for 循环中，会同时检查各行和各列，查看是否满足井字棋的获胜条件（即 3 个 X 或 O 排成一排）。可以通过 |= 完成此任务，这类似于 +=，但它使用了 or 运算符而不是加法运算符。然后，检查两个对角上是否包含相同的字符。最后，使用一些布尔代数运算来返回是否满足任何获胜条件。再编写 3 个方法，您就完成了此次编码。
您想再玩一次吗？

如果返回并查看 清单 4 中的 play_game 方法，就会看到代码在一直循环，直到 finished 为 true。只有在方法 player_is_finished 为 true 时才会出现这种情况。此方法应该基于玩家的响应：yes 或 no (清单 14)。
清单 14. player_is_finished 方法

```
fn player_is_finished() -> bool {

    let mut player_input = String::new();

    println!("Are you finished playing (y/n)?:");

    match io::stdin().read_line(&mut player_input) {

        Ok(_) => {

            let temp = player_input.to_lowercase();

            temp.trim() == "y" || temp.trim() == "yes"

        }

            Err(_) => false,

    }

}
```

在最初编写此方法时，我断定最好的方法是仅处理玩家输入的“yes”情况，这意味着其他所有输入都返回 false。这同样是一个静态方法，因为它没有使用 self 带来的任何数据。
一次硬重置可以修复所有问题

play_game 中使用的最后一个方法是 reset，如清单 15 所示。
清单 15. reset 方法

```
fn reset(&mut self) {

    self.current_turn = Turn::Player;

    self.board = vec![

        vec![

            String::from("1"), String::from("2"),  

            String::from("3")],

        vec![

            String::from("4"), String::from("5"), 

            String::from("6")],

        vec![

            String::from("7"), String::from("8"), 

            String::from("9")],

    ];

}
```

此方法的唯一功能就是将游戏的成员变量设置回它们的默认值。

要完成游戏，还需要最后一个方法，即 get_next_turn，如清单 16 所示。
清单 16. get_next_turn 方法

```
fn get_next_turn(&self) -> Turn {

    match self.current_turn {

        Turn::Player => Turn::Bot,

        Turn::Bot => Turn::Player,

    }

}
```

此方法仅检查 self 是谁的轮次，并返回对方的轮次。
## 运行并编译游戏

完成 game.rs 模块后，您现在可以编译 main.rs 并玩游戏了 (清单 17)。
清单 17. 编译游戏
```
extern crate rand;

mod game;

use game::Game;

fn main() {

    println!("Welcome to Tic-Tac-Toe!");

    let mut game = Game::new();

    game.play_game();

}
```

就这么简单。您通过 mod 声明了游戏模块位于此项目中，通过 use 将 Game 对象引入作用域内。然后，您通过 Game::new() 创建了一个 game 对象，并告诉该对象玩此游戏。现在，通过 Cargo 运行它 (清单 18)。
清单 18. 运行游戏
```
$ cargo run

   Compiling tic_tac_toe v0.1.0 …

    Finished dev [unoptimized + debuginfo] …

     Running …

Welcome to Tic-Tac-Toe! 

+---+---+---+

| 1 | 2 | 3 |

+---+---+---+

| 4 | 5 | 6 |

+---+---+---+

| 7 | 8 | 9 |

+---+---+---+

Please enter your move (an integer between 1 and 9): 

……
```

## 最后的思考

学完整篇教程后就会知道，Rust 是一种通用语言，它既拥有 Java、C# 或 Python 的易用性，又拥有 C 或 C++ 的速度和强大功能。此代码不仅已编译并运行快速，而且所有内存和错误问题都已在编译时得到解决，而不是留到运行时，并减少了代码中可能的人为错误。 
