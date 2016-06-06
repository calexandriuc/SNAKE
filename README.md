'SNAKE CODE


'SNAKE PARTS === FILE

Public Class SnakePart
  Protected Friend mColor As Color
  Protected Friend mRect As Rectangle
  Public Property Color() As Color
    Get
      Return mColor
    End Get
    Set(ByVal value As Color)
      mColor = value
    End Set
  End Property
  Public Sub New()
        mColor = Drawing.Color.Green
    mRect = New Rectangle(0, 0, 10, 10)
  End Sub
  Public Sub New(ByVal c As Color, ByVal x As Integer, ByVal y As Integer)
    mColor = c
    mRect = New Rectangle(x, y, 10, 10)
  End Sub
End Class

'FOOD FOR SNAKE

Public Class Food
  Protected Friend mColor As Color
  Protected Friend mRect As Rectangle
  Public ReadOnly Property Color() As Color
    Get
      Return mColor
    End Get
  End Property
  Public ReadOnly Property Rect As Rectangle
    Get
      Return mRect
    End Get
  End Property

  Public Sub New(ByRef land As SnakeGame, ByVal c As Color, ByVal x As Integer, ByVal y As Integer)
    mColor = c
    mRect = New Rectangle(x + land.mBounds.Left, y + land.mBounds.Top, land.mSize, land.mSize)
  End Sub
  Public Sub Draw(ByRef g As Graphics)
    Using b As New Pen(mColor)
            g.FillEllipse(b.Brush, mRect)
            g.DrawEllipse(Pens.AliceBlue, mRect)
    End Using
  End Sub
End Class

'SNAKE SHAPE

Public Class Form2
    Public Event IsDead()
    Protected Friend mLand As SnakeGame
    Protected Friend mHead As Color = Color.Yellow
    Protected Friend mBody As Color = Color.Green
    Protected Friend mParts As New Queue(Of SnakePart)
    Protected Friend mHeading As MoveDirections = MoveDirections.East

    Public Event EatenFood(ByRef f As Food)

    Public Sub New(ByRef onLand As SnakeGame)
        mLand = onLand
        For i = 0 To 30 Step 10
            mParts.Enqueue(New SnakePart(Color.Green, i, 50))
        Next
        mParts.Last.Color = Color.Yellow
    End Sub
    Public Property Heading() As MoveDirections
        Get
            Return mHeading
        End Get
        Set(ByVal value As MoveDirections)
            If [Enum].IsDefined(GetType(MoveDirections), value) Then mHeading = value
        End Set
    End Property
    Public Sub Move(ByVal md As MoveDirections, Optional ByVal Lengthen As Boolean = False)
        If [Enum].IsDefined(GetType(MoveDirections), md) = False Then Throw New ArgumentException(String.Format("{0} is not a valid value.", md))
        mHeading = md
        If mParts.Count > 0 Then
            mParts.Last.Color = mBody
            Dim sp As SnakePart = Nothing
            Select Case md
                Case MoveDirections.North : sp = New SnakePart(Color.Yellow, mParts.Last.mRect.X, mParts.Last.mRect.Y - mLand.mSize)
                Case MoveDirections.East : sp = New SnakePart(Color.Yellow, mParts.Last.mRect.X + mLand.mSize, mParts.Last.mRect.Y)
                Case MoveDirections.South : sp = New SnakePart(Color.Yellow, mParts.Last.mRect.X, mParts.Last.mRect.Y + mLand.mSize)
                Case MoveDirections.West : sp = New SnakePart(Color.Yellow, mParts.Last.mRect.X - mLand.mSize, mParts.Last.mRect.Y)
            End Select
            Dim ft As IEnumerable(Of Food) = mLand.mFood.Where(Function(fp As Food) fp.Rect.IntersectsWith(sp.mRect))
            If ft.Any Then
                Lengthen = True
                RaiseEvent EatenFood(ft(0))
            End If
            If Not Lengthen Then mParts.Dequeue()
            Dim MoveIntoSelf = mParts.Any(Function(dd As SnakePart) dd.mRect.IntersectsWith(sp.mRect))
            ' Verifica daca snake este mort
            If MoveIntoSelf Then RaiseEvent IsDead()
            If sp.mRect.X < mLand.mBounds.Left Then RaiseEvent IsDead()
            If sp.mRect.X > mLand.mBounds.Right Then RaiseEvent IsDead()
            If sp.mRect.Y < mLand.mBounds.Top Then RaiseEvent IsDead()
            If sp.mRect.Y > mLand.mBounds.Bottom Then RaiseEvent IsDead()
            mParts.Enqueue(sp)
        End If
    End Sub
    Public Sub DrawSnake(ByRef g As Graphics)
        For Each sp As SnakePart In mParts
            Using b As New Pen(sp.Color)
                g.FillRectangle(b.Brush, sp.mRect)
                g.DrawRectangle(Pens.Black, sp.mRect)
            End Using
        Next
    End Sub

    Private Sub Form2_Load(sender As Object, e As EventArgs) Handles MyBase.Load

    End Sub

    Private Sub InitializeComponent()
        Me.SuspendLayout()
        '
        'Form2
        '
        Me.ClientSize = New System.Drawing.Size(284, 261)
        Me.Name = "Form2"
        Me.ResumeLayout(False)

    End Sub
End Class

'DIRECTIONS

Public Class DemoPlayer
  Protected Friend mGame As SnakeGame
  Protected Friend mRnd As New Random
  Protected Friend mDemoPlay As New System.Windows.Forms.Timer With {.Interval = 500, .Enabled = True}
  Public Sub New(ByRef g As SnakeGame)
    mGame = g
    AddHandler mDemoPlay.Tick, AddressOf TheDemoPlayer
    AddHandler mGame.GameOver, AddressOf GameOver
  End Sub
  Public Sub GameOver()
    mGame = New SnakeGame
  End Sub
  Private Sub TheDemoPlayer(ByVal sender As System.Object, ByVal e As System.EventArgs)
    Dim n = mRnd.NextDouble
    Select Case mGame.mSnake.Heading
      Case MoveDirections.North
        Select Case n
          Case n <= 0.3 : mGame.Move(MoveDirections.North)
          Case (n > 0.3) AndAlso (n <= 0.6) : mGame.Move(MoveDirections.East)
          Case (n > 0.6) AndAlso (n <= 0.9) : mGame.Move(MoveDirections.West)
          Case n > 0.9 : mGame.Move(MoveDirections.South)
        End Select
      Case MoveDirections.East
        Select Case True
          Case n <= 0.3 : mGame.Move(MoveDirections.East)
          Case (n > 0.3) AndAlso (n <= 0.6) : mGame.Move(MoveDirections.North)
          Case (n > 0.6) AndAlso (n <= 0.9) : mGame.Move(MoveDirections.South)
          Case n > 0.9 : mGame.Move(MoveDirections.West)
        End Select
      Case MoveDirections.South
        Select Case n
          Case n <= 0.3 : mGame.Move(MoveDirections.South)
          Case (n > 0.3) AndAlso (n <= 0.6) : mGame.Move(MoveDirections.East)
          Case (n > 0.6) AndAlso (n <= 0.9) : mGame.Move(MoveDirections.West)
          Case n > 0.9 : mGame.Move(MoveDirections.North)
        End Select
      Case MoveDirections.West
        Select Case n
          Case n <= 0.3 : mGame.Move(MoveDirections.West)
          Case (n > 0.3) AndAlso (n <= 0.6) : mGame.Move(MoveDirections.South)
          Case (n > 0.6) AndAlso (n <= 0.9) : mGame.Move(MoveDirections.North)
          Case n > 0.9 : mGame.Move(MoveDirections.East)
        End Select
    End Select
    My.Application.DoEvents()
  End Sub
End Class

'GAME STATE

Public Enum GameState As Integer
  GameOver = 0
  Playing = 1
  Paused = 2
End Enum

'Gaming States

Public Enum GamingStates As Integer
  Demo = 0
  Player = 1
End Enum

'MOVE DIRECTIONS

Public Enum MoveDirections As Integer
  North = 0
  East = 1
  South = 2
  West = 3
End Enum

'SNAKE GAME

Public Class SnakeGame
    Protected Friend mBounds As New Rectangle(0, 25, 350, 350)
  Protected Friend mSize As Integer = 10
  Protected Friend mRnd As New Random
    Protected Friend WithEvents mSnake As Form2
    Protected Friend mFood As New List(Of Food)
    Protected Friend mGameState As GameState
    Protected Friend mInitialFood As Integer = 3
    Protected Friend mPoints As Integer = 0
    Protected Friend mTick As New System.Windows.Forms.Timer With {.Interval = 500, .Enabled = True}
#Region "Events"
    Public Event GameOver()
    Public Event Updated()
#End Region
#Region "ReadOnly Properties"
    Public ReadOnly Property Points As Integer
        Get
            Return mPoints
        End Get
    End Property
    Public ReadOnly Property GameState
        Get
            Return mGameState
        End Get
    End Property
#End Region
    Private Sub Tick(ByVal sender As System.Object, ByVal e As System.EventArgs)
        If GameState = GameState.Playing Then Move()
    End Sub
    Public Sub New()
        'Initializeaza un Snake now
        mSnake = New Form2(Me)
        'Populeaza zona formei cu mancare
        For i = 1 To mInitialFood
            AddNewPieceOfFood()
        Next
        AddHandler mSnake.IsDead, AddressOf HandleDeadSnake
        AddHandler mSnake.EatenFood, AddressOf HasEatenFood
        mGameState = GameState.Playing
        AddHandler mTick.Tick, AddressOf Tick
    End Sub
  Public Sub Pause()
    If mGameState = GameState.Playing Then
      mGameState = GameState.Paused
    ElseIf mGameState = GameState.Paused Then
      mGameState = GameState.Playing
    End If
  End Sub
  Private Sub HasEatenFood(ByRef f As Food)
    mFood.Remove(f)
    mPoints += 1
    If mFood.Any = False Then AddNewPieceOfFood()
  End Sub
  Private Sub HandleDeadSnake() Handles mSnake.IsDead
        mGameState = GameState.GameOver
        MsgBox("Game over!", MsgBoxStyle.Exclamation)
    End Sub

  Public Sub Move()
    Move(mSnake.Heading)
  End Sub
  Public Sub Move(ByVal md As MoveDirections)
    mSnake.Move(md)
        If mRnd.NextDouble <= 0.02 Then AddNewPieceOfFood()
    RaiseEvent Updated()
  End Sub
  Private Sub AddNewPieceOfFood()
    Dim x As Integer = 0
    Dim y As Integer = 0
        Dim fr As Food = Nothing
    Dim OverFood As Boolean = True
    Dim OverSnake As Boolean = True
    Do
            x = mRnd.Next(0, 25)
            y = mRnd.Next(0, 25)
            fr = New Food(Me, Color.Blue, x * mSize, y * mSize)
      OverFood = mFood.Any(Function(f As Food) f.Rect.IntersectsWith(fr.Rect))
      OverSnake = mSnake.mParts.Any(Function(s As SnakePart) s.mRect.IntersectsWith(fr.Rect))
    Loop Until Not (OverFood OrElse OverSnake)
    mFood.Add(New Food(Me, fr.Color, fr.Rect.X, fr.Rect.Y))
  End Sub
  Public Sub Draw(ByRef g As Graphics)
    For Each fp As Food In mFood
      fp.Draw(g)
    Next
    mSnake.DrawSnake(g)
  End Sub
End Class

'
