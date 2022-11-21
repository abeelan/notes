

### Android/iOS 基础知识

Android 是通过容器的布局属性来管理子控件的位置关系，布局过程就是把界面上的所有控件根据间距大小摆放到正确的位置。

布局是一种可放置很多控件的容器，它可以按照一定得规律调整内部控件的位置，从而编写精美的界面。布局内部除了放置空间外，也可以放置布局，通过多布局的嵌套，能完成一些很复杂的界面。



七大布局：

- LinearLayout 线性布局
- RelativeLayout 相对布局
- FrameLayout 帧布局
- TableLayout 表格布局
- GridLayout 网格布局
- ConstraintLayout 约束布局



常用的控件：

- TextView（文本控件）、EditText（可编辑文本控件）
- Button（按钮）、ImageButton（图片按钮）、ToggleButton（开关按钮）
- ImageView（图片控件）
- CheckBox（复选框控件）、RadioButton（单选框控件）



Android 四大组件：

- activity：与用户交互的可视化界面
- service：实现程序后台运行的解决方案
- content provider：内容提供者，提供程序所需要的数据
- broadcast receiver：广播接收器，监听外部事件的到来（比如来电）