####  Golden Master Pattern ：一种在.NET Core中重构遗留代码的利器

在软件开发领域中工作的任何人都将需要在旧代码中添加功能，这些功能可能是从先前的团队继承而来的，您需要对其进行紧急修复。

可以在文献中找到许多遗留代码的定义，我更喜欢的定义是：“通过**遗留代码**，我们指的是我们**害怕改变**的有利可图的代码”。

该定义包含两个基本概念：

1. 该代码必须有利可图。如果不是这样，我们将无意对其进行更改。 
2. 它必须引起对修改它的恐惧，因为我们可以引入新的bug或依赖影子的东西。

在以下情况下，更容易出错：

- 测试未涵盖该代码。 
- 代码不干净；不遵守单一责任原则。 
- 该代码的设计不正确，或者随着时间的流逝其结构变得不合理：对一段代码进行更改可能会产生一些副作用。 
- 您没有时间全面了解正在修改的内容。

测试是我们作为开发人员可用的强大武器。这些测试为我们提供了结果的安全性，并且是一种快速检测错误的方法。但是，我们如何测试未知的代码？构建单元测试套件将为我们提供有关该项目的深入知识，但它将使长时间保持高成本。如果我们无法测试细节，则可以使用**Characterization Test**，它是描述软件行为的测试。

在这种情况下起重要作用的**模式**是“ **黄金大师模式”**。基本思想很简单：如果我们无法深入了解，我们需要一些有关整个执行过程的指标。我们捕获正确执行的输出（stdout，图像，日志文件等），这就是我们的Golden Master，可用于预期输出。如果当前执行的输出匹配，我们可以确信我们的更改没有引入新的错误。 

为了展示Golden Master Pattern的用法，让我们从一个示例开始（完整的代码可以在[此处](https://github.com/ntonjeta/GoldenMasterExample)找到）。我们公司开发了用于命令行的游戏，包括井字游戏（该游戏的实现从[此处获取](https://www.c-sharpcorner.com/UploadFile/75a48f/tic-tac-toe-game-in-C-Sharp/)），我们的老板要求我们更改游戏以提供调整游戏板尺寸的能力。让我们看一下代码：

```
namespace Tris
{
    public class Game
    {
        //making array and   
        //by default I am providing 0-9 where no use of zero  
        static char[] arr = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' };
        static int player = 1; //By default player 1 is set  
        static int choice; //This holds the choice at which position user want to mark   
        // The flag variable checks who has won if its value is 1 then someone has won the match if -1 then Match has Draw if 0 then match is still running  
        static int flag = 0;
 
        public static void run()
        {
            do
            {
                Console.Clear();// whenever loop will be again start then screen will be clear  
                Console.WriteLine("Player1:X and Player2:O");
                Console.WriteLine("\n");
                if (player % 2 == 0)//checking the chance of the player  
                {
                    Console.WriteLine("Player 2 Chance");
                }
                else
                {
                    Console.WriteLine("Player 1 Chance");
                }
 
                Console.WriteLine("\n");
                Board();// calling the board Function  
                choice = int.Parse(Console.ReadLine());//Taking users choice   
 
                // checking that position where user want to run is marked (with X or O) or not  
                if (arr[choice] != 'X' && arr[choice] != 'O')
                {
                    if (player % 2 == 0) //if chance is of player 2 then mark O else mark X  
                    {
                        arr[choice] = 'O';
                        player++;
                    }
                    else
                    {
                        arr[choice] = 'X';
                        player++;
                    }
                }
                else //If there is any position where user wants to run and that is already marked then show message and load board again  
                {
                    Console.WriteLine("Sorry the row {0} is already marked with {1}", choice, arr[choice]);
                    Console.WriteLine("\n");
                    Console.WriteLine("Please wait 2 second board is loading again.....");
                    Thread.Sleep(2000);
                }
                flag = CheckWin();// calling of check win  
            } while (flag != 1 && flag != -1);// This loof will be run until all cell of the grid is not marked with X and O or some player is not winner 
 
            Console.Clear();// clearing the console  
 
            Board();// getting filled board again  
 
            if (flag == 1)// if flag value is 1 then someone has win or means who played marked last time which has win  
            {
                Console.WriteLine("Player {0} has won", (player % 2) + 1);
            }
 
            else// if flag value is -1 the match will be drawn and no one is the winner  
            {
                Console.WriteLine("Draw");
            }
 
            Console.ReadLine();
        }
 
        // Board method which creats board  
        private static void Board()
        {
            Console.WriteLine("     |     |      ");
            Console.WriteLine("  {0}  |  {1}  |  {2}", arr[1], arr[2], arr[3]);
            Console.WriteLine("_____|_____|_____ ");
            Console.WriteLine("     |     |      ");
            Console.WriteLine("  {0}  |  {1}  |  {2}", arr[4], arr[5], arr[6]);
            Console.WriteLine("_____|_____|_____ ");
            Console.WriteLine("     |     |      ");
            Console.WriteLine("  {0}  |  {1}  |  {2}", arr[7], arr[8], arr[9]);
            Console.WriteLine("     |     |      ");
        }
 
        private static int CheckWin()
        {
            #region Horzontal Winning Condtion
            //Winning Condition For First Row   
            if (arr[1] == arr[2] && arr[2] == arr[3])
            {
                return 1;
            }
 
            //Winning Condition For Second Row   
            else if (arr[4] == arr[5] && arr[5] == arr[6])
            {
                return 1;
            }
 
            //Winning Condition For Third Row   
            else if (arr[6] == arr[7] && arr[7] == arr[8])
            {
                return 1;
            }
 
            #endregion
 
            #region vertical Winning Condtion
 
            //Winning Condition For First Column       
            else if (arr[1] == arr[4] && arr[4] == arr[7])
            {
                return 1;
            }
 
            //Winning Condition For Second Column  
            else if (arr[2] == arr[5] && arr[5] == arr[8])
            {
                return 1;
            }
 
            //Winning Condition For Third Column  
            else if (arr[3] == arr[6] && arr[6] == arr[9])
            {
                return 1;
            }
 
            #endregion
 
            #region Diagonal Winning Condition
            else if (arr[1] == arr[5] && arr[5] == arr[9])
            {
                return 1;
            }
 
            else if (arr[3] == arr[5] && arr[5] == arr[7])
            {
                return 1;
            }
 
            #endregion
 
            #region Checking For Draw
 
            // If all the cells or values filled with X or O then any player has won the match  
            else if (arr[1] != '1' && arr[2] != '2' && arr[3] != '3' && arr[4] != '4' && arr[5] != '5' && arr[6] != '6' && arr[7] != '7' && arr[8] != '8' && arr[9] != '9')
            {
                return -1;
            }
 
            #endregion
 
            else
            {
                return 0;
            }
        }
    }
}
```

快速阅读后，代码看上去很混乱，职责没有正确分开，变量名也没有意义。

经过准确的阅读后，我们可以找到游戏板，该游戏板存储在“ static char [] arr”中。向阵列添加新元素没有任何效果，因为该阵列直接在PrintBoard和CheckWin函数中访问。现在我们知道要调整游戏板的大小，必须更改大部分代码。 

创建一个新项目并运行游戏：

```
class Program
{
    static void Main(string[] args)
    {
        Game.run();
    }
}
```

 一旦我们印刷了棋盘，游戏就会要求用户输入。我们可以通过从文件中读取输入来实现自动化。 

```
class Program
{
    private const string InputPath = "input.txt";
 
    public static void Main(string[] args)
    {
        var input = new StreamReader(new FileStream(InputPath, FileMode.Open));
        Console.SetIn(input);
        Game.run();
        input.Close();
    }
}
```

所有输入的集合太大，无法使用蛮力测试。我们可以做的就是对输入进行**采样**。为此，我们考虑井字游戏的最终得分：

- 玩家1获胜
- 玩家2获胜
- 绘制图形

选择覆盖这三种情况的最低限度的测试集，在文本文件中编写路径，并在golendenMaster文件夹中收集结果：

```
class Program
{
    private const string InputFolderPath = "input/";
    private const string OutputFolderPath = "goldenMaster/";
 
    public static void Main(string[] args)
    {
        int i = 1;
        foreach (var filePath in Directory.GetFiles(InputFolderPath)) {
            var input = new StreamReader(new FileStream(filePath, FileMode.Open));
            var output = new StreamWriter(new FileStream(OutputFolderPath + "output" + i.ToString() + ".txt" , FileMode.CreateNew));
            Console.SetIn(input);
            Console.SetOut(output);
            Game.run();
            input.Close();
            output.Close();
            i++;
        }
    }
}
```

 这三个结果文件代表了我们的Golden Master，我们可以在此基础上进行一些特性测试： 

```
[Test]
public void WinPlayerOne()
{
    inputPath = InputFolderPath + "input1.txt";
    outputPath = OutputFolderPath + "output.txt";
    var goldenMasterOutput = GoldenMasterOutput + "output1.txt";
 
    var input = new StreamReader(new FileStream(inputPath, FileMode.Open));
    var output = new StreamWriter(new FileStream(outputPath, FileMode.CreateNew));
    Console.SetIn(input);
    Console.SetOut(output);
 
    Game.run();
 
    input.Close();
    output.Close();
 
    Assert.True(AreFileEquals(goldenMasterOutput, outputPath));
}
 
private bool AreFileEquals(string expectedPath, string actualPath)
{
    byte[] bytes1 = Encoding.Convert(Encoding.ASCII, Encoding.ASCII, Encoding.ASCII.GetBytes(File.ReadAllText(expectedPath)));
    byte[] bytes2 = Encoding.Convert(Encoding.ASCII, Encoding.ASCII, Encoding.ASCII.GetBytes(File.ReadAllText(actualPath)));
 
    return bytes1.SequenceEqual(bytes2);
}
```

 只要测试是绿色的，我们就可以重构而不必担心破坏某些东西。一种可能的结果可能是： 

```
public static void run()
{
    char[] board = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' };
    int actualPlayer = 1;
 
    while (CheckWin(board) == 0)
    {
        PrintPlayerChoise(actualPlayer);
        PrintBoard(board);
        var choice = ReadPlayerChoise();
        if (isBoardCellAlreadyTaken(board[choice]))
        {
            PrintCellIsAlreadyMarketMessage(board[choice], choice);
            continue;
        }
        board[choice] = GetPlayerMarker(actualPlayer);
        actualPlayer = UpdatePlayer(actualPlayer);
    }
 
    PrintResult(board, actualPlayer);
}
```

从这段代码中可以看出Board的概念及其职责。让我们尝试在新的**Board**类中提取行为。新Board应能够： 

- 印刷板
- 标记玩家的选择 
- 检查是否有赢家

使用TDD（更多详情，请阅读[这篇文章](https://www.blexin.com/en-US/Article/Blog/TDD-the-whole-code-is-guilty-until-proven-innocent-38)）制定一个可调整大小的Board（发现测试的完整代码[在这里](https://github.com/ntonjeta/GoldenMasterExample/blob/master/test/GoldenMasterExampleTest/BoardShould.cs)和阶级的一个[位置](https://github.com/ntonjeta/GoldenMasterExample/blob/master/lib/Game/Board.cs)）。现在尝试将它们插入游戏中，并检查Golden Master是否保持绿色：

```
private const int Boardsize = 3;
 
public static void run()
{
    Board board = new Board(Boardsize);
    int actualPlayer = 1;
 
    while (board.CheckWin() == -1)
    {
        PrintPlayerChoise(actualPlayer);
        Console.WriteLine(board.Print());
        var choice = ReadPlayerChoise();
        if (!board.UpdateBoard(actualPlayer, choice))
        {
            PrintCellIsAlreadyMarketMessage(board.GetCellValue(choice), choice);
            continue;
        }
        actualPlayer = UpdatePlayer(actualPlayer);
    }
 
    PrintResult(board, actualPlayer);
}
```

 此时，我们可以恢复标准输入/标准输出并从用户那里读取电路板的尺寸： 

```
class Program
{
    public static void Main(string[] args)
    {
        Console.WriteLine("Insert Diagonal dimension of Board: ");
        var boardSize = int.Parse(Console.ReadLine());
        Game.run(boardSize);
    }
}
```

如您所见，多亏了Golden Master Pattern，我们能够控制遗留代码并进行重构，而无需担心。但是，所有闪闪发光的东西都不是金子：在“噪声输出”的情况下使用Golden Master可能会很困难，“噪声输出”对于执行无用，但会随时间（例如时间戳记，线程名等）而变化。在这种情况下，我们可以过滤输出并仅考虑重要部分。

我希望它在您下次重写旧项目时对您有用：毕竟，我们担心我们的代码失去控制！