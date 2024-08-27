目录  
[一、介绍](#1)  
[二、创建多人游戏插件](#2)  
[三、创建项目](#3)  
&emsp;[3.1 项目创建及准备工作](#3.1)  
&emsp;[3.2 资产 Assets](#3.2)  
&emsp;[3.3 动画及重定向 Retargeting Animations](#3.3)  
&emsp;[3.4 角色创建](#3.4)  
&emsp;[3.5 摄像机和弹簧臂 Camera and Spring Arm](#3.5)  
&emsp;[3.6 人物移动](#3.6)  
&emsp;[3.7 动画蓝图](#3.7)  
&emsp;[3.8 无缝传送 Seamless travel and lobby](#3.8)  
&emsp;[3.9 网络规则 Network Role](#3.9)  
[四、武器](#4)  
&emsp;[4.1 武器类](#4.1)  
&emsp;[4.2 捡子弹 Pickup Widget](#4.2)  
&emsp;[4.3 变量复制 Variable Replication](#4.3)  
&emsp;[4.4 装备武器 Equipping Weapons](#4.4)  
&emsp;[4.5 远程过程调用 Remote Procedure Calls](#4.5)  
&emsp;[4.6 装备动画姿势 Equipped Animation Pose](#4.6)  
&emsp;[4.7 蹲下 Crouching](#4.7)  
&emsp;[4.8 瞄准 Aiming](#4.8)  
&emsp;[4.9 跑步混合空间 Running Blendspace](#4.9)  
&emsp;[4.10 倾斜和扫射 Leaning and Strafing](#4.10)  
&emsp;[4.11 走路和跳跃 Idle and Jumps](#4.11)  
&emsp;[4.12 蹲走 Crouch Walking](#4.12)  
&emsp;[4.13 瞄准走 Aim Walking](#4.13)  
&emsp;[4.14 瞄准偏移 Aim Offsets](#4.14)  
&emsp;[4.15 瞄准偏移应用](#4.15)  
&emsp;[4.16 多人匹配 Pitch in Multiplayer](#4.16)  
&emsp;[4.17 使用瞄准偏移](#4.17)  
&emsp;[4.18 FABRIK IK](#4.18)  
&emsp;[4.19 原地转弯 Turning in place](#4.19)  
&emsp;[4.20 旋转根骨骼 Rotate Root Bone](#4.20)  
&emsp;[4.21 网络更新频率](#4.21)  
&emsp;[4.22 未装备时蹲着](#4.22)  
&emsp;[4.23 跑步转弯动画 Rotating Running Animations](#4.23)  
&emsp;[4.24 脚步声和跳跃声 Footstep and Jump Sounds](#4.24)  
&emsp;[4.25 瞄准走](#4.25)  
[五、射击武器](#5)  
[六、武器瞄准机制](#6)  
[七、健康状况和玩家统计数据](#7)  
[八、弹药](#8)  
[九、匹配状态](#9)  
[十、不同武器类型](#10)  
[十一、拾取](#11)  
[十二、滞后补偿](#12)  
[十三、更多多人游戏类型](#13)  
[十四、队伍](#14)  
[十五、夺旗](#15)  




**目前遇到的问题**

1. 3.7人物移动
root component和controller的方向可能不一样
具体有什么区别？
代码中，将yaw经过什么样的转换得到Direction？
得到的Direction是什么样的？
GetUnitAxis(EAxis::X)中XY的区别？







**UE知识**

1. 常见类
2. ![alt text](assets/blaster/image.png)
Object-Actor 







---

<span id = "1">

**一、介绍**

123123123





---

<span id = "2">

**二、创建多人游戏插件**





---

<span id = "3">

**三、创建项目**




<span id = "3.6">

**3.6 人物移动**

键盘映射
人物移动函数

1. 设置键盘映射
![](https://img2024.cnblogs.com/blog/2082701/202408/2082701-20240805010919638-143714592.png)

1. 代码
<details><summary>BlasterCharacter.h</summary>

```cpp
protected:
    void MoveForward(float Value);
    void MoveRight(float Value);
    void Turn(float Value);
    void LookUp(float Value);
```
</details>



<details><summary>BlasterCharacter.cpp</summary>

```cpp
void ABlasterCharacter::MoveForward(float Value)
{
	if (Controller != nullptr && Value != 0) {
		//const FVector Direction = GetActorForwardVector()  此函数返回角色向前向量，而我们需要改变我们控制器的前进
		const FRotator YawRotation(0.f, Controller->GetControlRotation().Yaw, 0.f);
		const FVector Direction(FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X));
		//得到一个平行于地面的向量，指向偏航的方向
		//const FVector Direction(FRotationMatrix(0.f, Controller->GetContrlRotation().Yaw, 0.f).GetUnitAxis(EAxis::X));
		AddMovementInput(Direction, Value);
	}
}

void ABlasterCharacter::MoveRight(float Value)
{
	//const FVector Direction = GetActorForwardVector()  此函数返回角色向前向量，而我们需要改变我们控制器的前进
	const FRotator YawRotation(0.f, Controller->GetControlRotation().Yaw, 0.f);
	const FVector Direction(FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y));
	//const FVector Direction(FRotationMatrix(0.f, Controller->GetContrlRotation().Yaw, 0.f).GetUnitAxis(EAxis::X));
	AddMovementInput(Direction, Value);
}

void ABlasterCharacter::Turn(float Value)
{
	AddControllerYawInput(Value);
}

void ABlasterCharacter::LookUp(float Value)
{
	AddControllerPitchInput(Value);
}

void ABlasterCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);

	PlayerInputComponent->BindAxis("MoveForward", this, &ABlasterCharacter::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &ABlasterCharacter::MoveRight);
	PlayerInputComponent->BindAxis("Turn", this, &ABlasterCharacter::Turn);
	PlayerInputComponent->BindAxis("LookUp", this, &ABlasterCharacter::LookUp);

}
```
</details>

3. 细节知识

* 欧拉角
roll---pitch---yaw
分别绕x y z旋转角度
翻滚 俯仰 偏航
代表了绕轴旋转的角度
参考：[欧拉角](https://blog.csdn.net/daidi1989/article/details/95167676)

* GetActorForwardVector()
该函数得到 character 的前进方向，也就是根节点 capsule 的 forward vector
但是我们要改变controller的方向，root component和controller的方向可能不一样

* AddMovementInput()
这个函数会根据第一个参数的值去移动角色，第二个参数val是个浮点数
val就是键盘映射那里的scale的值

* FRotator
Yaw 表示 摇头 就是绕 Z 移动；
Pich 表示 点头 就是绕 Y 移动；
Roll 你可以想象成左右晃脑，绕 X轴运动。
这三个变量组成了 UE4 里面所有物体的旋转方向，它是一个结构体叫 FRotator。




**3.7 动画蓝图**








---

<span id = "4">

**四、武器**





---

<span id = "5">

**五、射击武器**





---

<span id = "6">

**六、武器瞄准机制**





---

<span id = "7">

**七、健康状况和玩家统计数据**




---

<span id = "8">

**八、弹药**





---

<span id = "9">

**九、匹配状态**




---

<span id = "10">

**十、不同武器类型**





---

<span id = "11">

**十一、拾取**





---

<span id = "12">

**十二、滞后补偿**





---

<span id = "13">

**十三、更多多人游戏类型**





---

<span id = "14">

**十四、队伍**





---

<span id = "15">

**十五、夺旗**






