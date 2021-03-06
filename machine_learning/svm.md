​
前提——了解基本二分类问题、有高数和线性代数的基本知识。

0.支持向量机的思想

就一点——最大化分类间隔

在二分类中，我们的目标仅仅分开被标记的不同的两个类别，但是可以分开的线有无数条，如下图1所示，分开两类数据可能有无数条线，其中给出了三条，但这三条那一条更好呢？很显然是绿色的（因为它与正负样本保持足够的距离，距离越大意味着正确分类的概率越大）



   图1

解决SVM的难点只有一个那就是解决二次规划问题，但是幸运的是可以不怎么费劲的找到一些解决二次规划的库函数之类的下面简单的介绍一下二次规划问题。

二次规划

二次规划的一般形式如下所示：

min{\(\frac{1}{2}u^TQu+p^Tu\)};

s.t. \(a^{T}_{m}u<=c_m\) (m=1,2,...,M)

如果满足上述形式那么我们可以使用二次规划的库函数求解，至于如何求解二次规划问题（想知道可以看看参考2）不做过多叙述，下来再来简单的介绍库函数的常用形式

u=QP(Q,p,A,c);

其中 Q、p对应基本形式中的Q和p，A是 \(a^{T}_{m}\)组成的矩阵，c是\(c_{m}\)组成的矩阵

1.线性可分支持向量机

    所谓线性可分简言之就是使用直线（二维空间）、或者平面（三维）或者超平面（高维空间）可以将正负样本分开（以下统一称为超平面）。

    假设我使用的超平面公式是 w*x+b=0,那么SVM的目标是什么？就是寻找使得距离超平面最近的点和超平面的距离最大（就说你看着这句话晕不晕），那就更简单的解释，如图2所示，绿色箭头的距离显然比黑色箭头距离更远而绿色的线显然分类正确的概率比黑色线更大，找到这种距离最大的线，就是SVM的目标。



（为了方便这里将正样本标记为1，负样本标记为-1），根据点到平面的距离公式可知某个点xi 到超平面距离的公式为：d = |w*xi+b|/||w||（等价为：yi（w*xi+b）/||w||） ,有人给d起了一个好听的名字叫做“几何间隔”，那么假设距离最近的点为(x,y) ，那么可以将SVM表述为:max{y(w*x+b)/||w|| },但是仅仅这样是不够的，因为我们还要保证正确的分类，又因为我们暂时只考虑线性可分，所以正确分类可以形式化的表述为yi*(w*xi+b)/||w||>=d 。

可以将SVM表示为下面这个约束最优化问题：

 \(\max\frac{y(w*x+b)}{||w||}\)

s.t.  \(\frac{y_i(w*x_i+b)}{||w||}\) >=d (i=1,2,...N)

为了进一步化简公式，设r=y(w*x+b), (r也有一个好听的名字叫做“函数间隔”)，所以可以将上述公式化简为：

\(\max\frac{r}{||w||}\)

s.t. yi*(w*xi+b)>=r (i=1,2,...N)

由于2w*x+2b=0 同样是所求超平面的方程，故将2w和2b带入同样可以求得超平面，所以r的值并不影响SVM求解，那么令r=1；于是得到：

\(\max\frac{1}{||w||}\)

s.t. yi*(w*xi+b)>=1  (i=1,2,...N)

（||w|| 叫做向量w的长度）||w||=\(\sqrt{w^{T}w}\),所以最大化1/||w|| 相当于最小化||w||,为了避免根式我们求\(min\frac{1}{2}w^{T}w\)故可化简为：

 \(\min\frac{1}{2}w^{T}w\)

s.t. yi*(w*xi+b)>=1  (i=1,2,...N)

上边的公式是不是很熟悉？对二次规划，接下来只需要将约束条件改写成小于等于的形式，然后调用函数就可以解决问题了，需要注意的是二次规划中的u对应w和b。

下边是matlab主要代码：

data = load('ex2data1.txt');
X = data(:,[1,2]);
    Y = data(:,3);
[N,~] =size(X); 
H = [0,0,0;
     0,1,0;
     0,0,1];
f = [0;0;0];

a = -1*[Y.*ones(N,1),Y .* X(:,1),Y .*X(:,2)];
b = -1*ones(N,1);%小于等于-1

X1= quadprog(H,f,a,b);
[f1,f2] = plotData(X,Y);
hold on;
A = X1(2);
B = X1(3);
C = X1(1);
x=25:1:95;
y=(-A.*x-C)./B;
plot(x,y);
%绘制支持向量
T = Y .* (A*X(:,1)+B*X(:,2)+C);
T = T-1;
index = find(T<1e-6);
f3 = plot(X(index, 1), X(index, 2), 'ms','MarkerSize', 10,'LineWidth',2);
%绘制说明
xlabel('x1');
ylabel('x2');
legend([f1,f2,f3],'正样本','负样本','支持向量');
hold off;

结果：



(注：yi*(w*xi+b)==1 的实例点 （\(x_i,y_i\)）叫做支持向量）

拉格朗日对偶性问题

为了方便后续的SVM推导这里简单的介绍一下拉格朗日对偶性问题

对于条件极值：\[\min {f(x)}\] \[s.t. c_i(x)\leq 0,i=1,2,...,k\] \[h_j(x)=0,j=1,2,...,l\]

我们可以引入拉格朗日函数\(L(x,\alpha,\beta)= f(x)+\sum_{i=1}^{k}\alpha_{i}c_{i}(x)+\sum_{j=1}^{l}\beta_{j}h_{j}(x) \) ,\(\alpha_{i}\geq0\)并且当\(\alpha_i*c_i(x)=0\)。求极值,就能得到\(\min \limits_{x} \max \limits_{\alpha \geq0,\beta}L(x,\alpha,\beta)\)即可得条件极值的解。但是这个问题并不好解决,又因为\(\min \limits_{x} \max \limits_{\alpha \geq0,\beta}L\geq\max \limits_{\alpha \geq0,\beta} \min\limits_{x}L\),不等号右边的公式内部对\(\alpha\)和 \(\beta\)没有条件，所以右边的公式更好解决。于是有人证明了满足1.L是凸函数；2.这个问题有解；3.线性条件。则不等号两边同解。

【注—关于minmaxL】h(x)=0, c(x)<=0，现在是取\(L(x,\alpha,\beta)\)的最大值，\(\alpha\)*c(x)是<=0，所以L(a,b,x)只有在\(\alpha\)*g(x) = 0的情况下才能取得最大值，否则，就不满足约束条件，因此max{\(\alpha,\beta\)} \(L(x,\alpha,\beta)\)在满足约束条件的情况下就是f(x)。

具体来说，对于我们上面得到的公式，记\(L= \frac{1}{2}w^{T}w+\sum_{i=1}^{N}\alpha_{i}(1-y_{i}*(w*x_i+b))\)，求\(\max \limits_{\alpha \geq0} \min\limits_{b，w}L\)的解，

化解：

由\(\frac{\partial L}{\partial b}\)=0（极值点的必要条件，具体参见高等数学）,可得:\(-\sum_{i}^{N}\alpha_{i}y_{i}=0\) ,然后将其带回，所以我们只需求得\(\max\min\frac{1}{2}w^{T}w+\sum_{i=1}^{N}\alpha_{i}(1-y_{i}*(w*x_i))\),

同样 \(\frac{\partial L}{\partial w_j}\)=0,可得:\(w_j-\sum_{i}^{N}\alpha_{i}y_{i}x_{i,j}=0\) ,可得\(w=\sum_{i}^{N}\alpha_{i}y_{i}x_{i}=0\) ,带回原式得

\(\max\limits_{\alpha}\min\limits_{w}[-\frac{1}{2}w^{T}w+\sum_{k=1}^{N}\alpha_{k}]\)=\(\max\limits_{\alpha}[-\frac{1}{2}\sum_{i}^{N}\sum_{j}^{N}\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}x_{j}+\sum_{k}^{N}\alpha_{k}]\)

并且满足（KKT条件）:

 \[y_i(w*x_{i}+b)\geq1\]

\[\alpha_{i}\geq0\]

\[\alpha_{i}(1-y_{i}*(w*x_i+b))=0\]

\[\sum_{i}^{N}\alpha_{i}y_{i}=0,w=\sum_{i}^{N}\alpha_{i}y_{i}x_{i}\]

求解\(\alpha\)：

求解\(\alpha\)不难想到使用二次规划求解，于是将上述问题表示为：

 \(\min\limits_{\alpha}[\frac{1}{2}\sum_{i}^{N}\sum_{j}^{N}\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}x_{j}-\sum_{k}^{N}\alpha_{k}]\)

s.t. \(\alpha_{i}\geq0\)    

\[\sum_{i}^{N}\alpha_{i}y_{i}=0\]

求解w,b：

利用条件\(w=\sum_{i}^{N}\alpha_{i}y_{i}x_{i}\)可求解w,利用条件\(\alpha_{i}(1-y_{i}*(w*x_i+b))=0\),取\(\alpha_{i}>0\)，则\(b=(1-y_{i}*(w*x_{i}))/y_{i}=y_{i}-w*x_{i}\), (因为y只能取正负1)，所以可求得w,b。另外，当\(\alpha_{i}>0\)时，可知此时取得的点为支持向量（因为：\(y_{i}*(w*x_i+b)=1\)）。

非线性支持向量机

线性不可分，但是X映射到Z空间后线性可分（ Z=\(\phi(X)\) ），那么只需将上述推导中的X替换成Z即可。

\(\min\limits_{\alpha}[\frac{1}{2}\sum_{i}^{N}\sum_{j}^{N}\alpha_{i}\alpha_{j}y_{i}y_{j}z_{i}z_{j}-\sum_{k}^{N}\alpha_{k}]\)

则，Q(i,j) = \(y_{i}y_{j}z^T_{i}z_{j}\)

但是如此变化容易遇到的问题是Z空间维度太大，导致二次规划不好做！！！

2.核技巧

假设\(X^T= (x^{(1)},x^{(2)},...x^{(n)})\),变换（二维变换）后的\(Z^T=(1, x^{(1)}, x^{(2)}, . . . , x^{(n)} , (x^{(1)})^{2}, x^{(1)} x^{(2)} , . . . , x^{(1)} x^{(n)} , x^{(2)} x^{(1)} , (x^{(2)}) ^2, . . . , x^{(2)} x^{(n)}  , . . . , (x^{(n)})^{2} )\)

\(Z^TZ=\phi(x)^{T}\phi(x') = 1+\sum_{k=1}^{n}x^{(k)}x'^{(k)}+\sum_{i=1}^{n}\sum_{j=1}^{n}x^{(i)}x^{(j)}x'^{(i)}x'^{(j)}\)

\(=1+\sum_{k=1}^{n}x^{(k)}x'^{(k)}+\sum_{i=1}^{n}x^{(i)}x'x^{(i)}\sum_{j=1}^{n}x^{(j)}x'^{(j)}\)

 \(= 1+x^Tx'+(x^Tx')(x^Tx')\)

令\(K(x,x') = 1+x^Tx'+(x^Tx')(x^Tx')\),则Q(i,j) = \(y_{i}y_{j}K(x_i,x_j)\)。通过K来计算Q，往往比直接计算更快，K称为核函数（ kernel  function）。

计算b和超平面:

\(\alpha_{j}\ge0\),则b=\(y_{j}-w*z_{j}\)，又因为\(w=\sum_{i}^{N}\alpha_{i}y_{i}z_{i}\)，可知:

b =  \(y_{j}-(\sum_{i}^{N}\alpha_{i}y_{i}z_{i})^T*z_{j} = y_{j}-\sum_{i}^{N}\alpha_{i}y_{i}K(x_{i},x_{j}) \)

w好像无法通过K计算，但是我们计算w,b的目的就是计算分类超平面的公式，是否可以直接计算超平面公式？(记超平面为g)

\(g=sign(w^TZ+b=(\sum_{i}^{N}\alpha_{i}y_{i}z_{i})^TZ+b)=sign(\sum_{i}^{N}\alpha_{i}y_{i}K(x_{i},X)+b)\)  

其中，\(x_{i}\)为第i个实例点，sign(x)为符号函数。

常见的核函数：

线性核函数：\(X^TX+p\)

多项式核函数：\((\alpha+\beta X^TX)^p\);

高斯核函数：\(e^{-p||x-x'||^2}\)

需要注意的是：

 高斯核函数是一个X到无限多维空间的映射（通过泰勒公式可以推导出来）。
使用不恰当的参数容易造成过拟合的问题。
下面是matlab代码：

type = 3;%2对应多项式核函数 3对应高斯核函数
p = 0.001;% kernel参数
data = load('ex2data1.txt');
X = data(:,[1,2]);
Y = data(:,3);
[len,~] =size(X);


H = 0.5*((Y*Y').* kernel(X,X,type,p));
f = -ones(len,1);
A = [];
B = [];
Aeq = Y';
beq = 0;
[Alphas,fval,exitflag] = quadprog(H,f,A,B,Aeq,beq,zeros(len,1));
[f1,f2] = plotData(X,Y);


sv_indexs = find(Alphas>sqrt(eps(8)));%找大于0的alpha


%求b的值
Alpha_Temp  = repmat(Alphas(sv_indexs),1,length(sv_indexs));
Y_Temp  = repmat(Y(sv_indexs),1,length(sv_indexs));
b = mean(Y(sv_indexs)' - sum(Alpha_Temp.* Y_Temp .* kernel(X(sv_indexs,:),X(sv_indexs,:),type,p),1));


%画曲线的等高线
[x1,x2] = meshgrid(0:0.1:100,0:0.1:100);
[M,N] = size(x1);    
Length = M*N;                    
Datas = [reshape(x1,1,Length);reshape(x2,1,Length)]; 
result = predict(Alphas(sv_indexs), Y(sv_indexs), kernel(X(sv_indexs,:), Datas',type,p),b);


Z = reshape(result,[M,N]);
hold on;
contour(x1,x2,Z,[0,0],'g');%画等高线

%绘制支持向量
f3 = plot(X(sv_indexs, 1), X(sv_indexs, 2), 'ms','MarkerSize', 10,'LineWidth',1.5);
%绘制说明
xlabel('x1');
ylabel('x2');
legend([f1,f2,f3],'正样本','负样本','支持向量');
hold off

kernel函数代码：

function K = kernel(X,Y,type,C)  
K = X*Y';
if(nargin>=3)
   switch type
       case 2
           a= C(1);
           b=C(2);
           p=C(3); 
           K = (b*K+a).^p;
       case 3
            X2 = sum(X.*X,2);%每一行求和
            Y2 = sum(Y.*Y,2);  
            XY = X*Y';  
            K = -C*abs(repmat(X2,[1 size(Y2,1)]) + repmat(Y2',[size(X2,1) 1]) - 2*XY);  
            K = exp(K);
   end
end
    
end  
结果：



3.软间隔最大化

经过上边的过程，我们虽然可以得到一些不错的结果，但是都是基于两类样本点可分的情况下，若两类样本点不可分呢？比如有一部分噪声点（可以 理解为 错误值 或 被某些无法预知的外在因素影响的值）

事实上我们可以适当的放松要求，无需对所有的样本点都正确分类。此时的处理方法是引入变量\(err_i\ge0\),表示允许第i个点犯的错误的大小，将约束条件 \(y_i(w*x_{i}+b)\geq1\)变更为 \(y_i(w*x_{i}+b)\geq1-err_i\),同时要让整体犯的错误尽可能小并且间隔尽可能大，故需：\(\min[\frac{1}{2}w^{T}w+C\sum\limits_{i=1}\limits^{N}err_{i}]\); (此时的间隔称为软间隔)。

所以我们的问题又回到了以前的问题:

\(\min[\frac{1}{2}w^{T}w+C\sum\limits_{i=1}\limits^{N}err_{i}]\)

s.t. \(y_i(w*x_{i}+b)\geq1-err_i\),

      \(err_i\ge0\),

拉格朗日函数,其中\(\alpha \ge0,\beta \ge0\):

\(L(w,b,err,\alpha,\beta) = \frac{1}{2}w^{T}w+C\sum\limits_{i=1}\limits^{N}err_{i}+\sum_{j=1}^{N}\alpha_{j}(1-err_{j}-y_{j}*(w*x_j+b))+\sum_{k=1}^{N}\beta_{k}(-err_k)\)

求解：

(1)    \(\frac{\partial L}{\partial err_{i}}=C-\alpha_{i}-\beta_{i}=0\),

(2)    \(\frac{\partial L}{\partial b}=-\sum_{i}^{N}\alpha_{i}y_{i}=0\),

(3)    \(\frac{\partial L}{\partial w_{j}}= w_{j}-\sum_{i}^{N}\alpha_{i}y_{i}x_{i,j}=0\)。

由（1）式可知\(\alpha_{i}+\beta_{i}=C\),并且\(\beta_{i}\ge0\)，故\(0\leq \alpha_{i} \leq{C}\)，把\(\beta_{i}=C-\alpha{i}\)带回L可以得到\(L= \frac{1}{2}w^{T}w+\sum_{i=1}^{N}\alpha_{i}(1-y_{i}*(w*x_i+b))\)，哇塞得到了和以前推导的一样的公式，于是直接跳过之后两步不过分吧， 于是得到：

\(\min\limits_{\alpha}[\frac{1}{2}\sum_{i}^{N}\sum_{j}^{N}\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}x_{j}-\sum_{k}^{N}\alpha_{k}]\)

KKT条件: 

\(y_i(w*x_{i}+b)\geq1\)

\(0\leq \alpha_{i} \leq{C}\)

\(\alpha_{i}(1-err_{i}-y_{i}*(w*x_i+b))=0\) 

\(\beta{i}(-err_{i})=0\) (与上一个条件类似，均为拉格朗日对偶问题特定的条件)

\(\sum_{i}^{N}\alpha_{i}y_{i}=0,w=\sum_{i}^{N}\alpha_{i}y_{i}x_{i}\)

利用二次规划求得\(\alpha\)后接下来求解b,同样取一个\(\alpha_{i}\>0\),那么\(1-err_{i}-y_{i}*(w*x_i+b))=0\) ,又因为\(\beta{i}(-err_{i})=0 \iff (C-\alpha_{i})err_{i}=0\),若存在\(0< \alpha_{i} <C\),则取这样的\(\alpha_{i}\)，解出b,利用前边的解可得w。若不存在，则b只能由一些不等式约束b的范围，找一个可能的b值并给出w值。

同样的利用带核函数的计算公式也能得到w和b的解。

最后的代码不给出，可以去这里下载。

给出结果，可以看出容许了一个错误：



参考：

        [1] 约束规划问题与凸二次规划

        [2] 数值优化学习系列-二次规划

        [3] 拉格朗日乘子法与KKT条件详解

        [4] 机器学习技法

        [5] 统计学习方法(李航)

​