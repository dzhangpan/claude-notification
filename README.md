# Claude Code Notification Popup

A borderless animated notification that appears on your Windows desktop when Claude Code finishes a task. Features the Claude Code mascot doing a jumping animation in a dark rounded popup at the bottom-right of your screen.

![Claude Code notification popup demo](notification.gif)

---

## What it does

- Spawns a 260×290 borderless popup at the bottom-right corner of your screen
- Shows an animated blocky character (the Claude Code mascot) jumping, waving, and squashing
- Click to dismiss, or it auto-closes after 7 seconds
- Only fires when Claude explicitly decides to notify you — not on every response

---

## How it works

Claude Code has a **Stop hook** — a shell command that runs every time Claude finishes a response. The trick is a flag file: Claude creates `$TEMP/claude_notify.flag` when it wants to notify you. The hook checks for that file on every stop, and if it exists, launches the popup.

```
Claude finishes → Stop hook runs → flag file exists? → launch popup → delete flag
```

This way Claude can selectively notify you only when something meaningful is done, not after every single message.

---

## Requirements

- Windows 10/11
- PowerShell 5+ (built-in on Windows)
- [Claude Code CLI](https://claude.ai/code) installed

---

## Setup

### 1. Save the script

Create a file called `notify.ps1` anywhere on your machine. Suggested path:

```
C:\Users\<YourName>\.claude\notify.ps1
```

Paste the full script below into that file.

### 2. Add the Stop hook

Open (or create) `~/.claude/settings.json` and add the following. Replace the path with wherever you saved `notify.ps1`:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "[ -f \"$TEMP/claude_notify.flag\" ] && rm \"$TEMP/claude_notify.flag\" && powershell.exe -ExecutionPolicy Bypass -Command \"Start-Process powershell -WindowStyle Hidden -ArgumentList '-ExecutionPolicy Bypass -File C:\\\\Users\\\\<YourName>\\\\Desktop\\\\notify.ps1'\"; exit 0"
          }
        ]
      }
    ]
  }
}
```

> If you already have other settings in that file, just merge the `hooks` section in — don't overwrite the whole file.

### 3. Tell Claude when to notify you

At the end of any response where you want a notification, Claude needs to run:

```bash
touch "$TEMP/claude_notify.flag"
```

You can ask Claude directly: *"notify me when you're done"* or *"set the notification flag at the end"*. Claude will create the flag and the popup will fire when it finishes.

---

## The script (`notify.ps1`)

```powershell
Add-Type -AssemblyName PresentationFramework
Add-Type -AssemblyName PresentationCore
Add-Type -AssemblyName WindowsBase

$xaml = @"
<Window
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  WindowStyle="None" AllowsTransparency="True" Background="Transparent"
  Topmost="True" ShowInTaskbar="False"
  Width="260" Height="290" ResizeMode="NoResize">
  <Window.Resources>
    <Storyboard x:Key="Anim" RepeatBehavior="Forever">

      <!-- JUMP -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="CharGroup"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <DiscreteDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:0.48" Value="0"/>
        <SplineDoubleKeyFrame   KeyTime="0:0:0.96" Value="-65" KeySpline="0.33,1 0.68,1"/>
        <SplineDoubleKeyFrame   KeyTime="0:0:1.48" Value="0"   KeySpline="0.32,0 0.67,0"/>
        <SplineDoubleKeyFrame   KeyTime="0:0:1.8"  Value="-22.5" KeySpline="0.33,1 0.68,1"/>
        <SplineDoubleKeyFrame   KeyTime="0:0:2.08" Value="0"   KeySpline="0.32,0 0.67,0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"    Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- SQUASH ScaleX -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="CharGroup"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)">
        <DiscreteDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.42" Value="1.25"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.54" Value="0.75"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.96" Value="0.95"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.4"  Value="1.35"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.48" Value="0.85"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.8"  Value="0.9"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.08" Value="1.2"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.32" Value="0.95"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.6"  Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"    Value="1"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- SQUASH ScaleY -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="CharGroup"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)">
        <DiscreteDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.42" Value="0.75"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.54" Value="1.3"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:0.96" Value="1.05"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.4"  Value="0.65"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.48" Value="1.2"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:1.8"  Value="1.1"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.08" Value="0.8"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.32" Value="1.05"/>
        <LinearDoubleKeyFrame   KeyTime="0:0:2.6"  Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"    Value="1"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- SHADOW ScaleX -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Shadow"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleX)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="1.3"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.54" Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="0.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.4"  Value="0.9"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="1.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="0.7"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="1.1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="1"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Shadow"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(ScaleTransform.ScaleY)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="1.3"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.54" Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="0.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.4"  Value="0.9"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="1.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="0.7"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="1.1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="1"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Shadow"
        Storyboard.TargetProperty="Opacity">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="0.8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.54" Value="0.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="0.2"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.4"  Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0.8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="0.4"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="0.7"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0.6"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0.6"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- BODY TILT -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="BodyTilt"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(RotateTransform.Angle)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="-3"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="-2"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- FACE LOOK RIGHT -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="FaceShift"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.X)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- ARM LEFT -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="ArmLeft"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(RotateTransform.Angle)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-45"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.84" Value="-85"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="-45"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.08" Value="-85"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.2"  Value="-45"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.32" Value="-85"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="-45"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.68" Value="-85"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.88" Value="-45"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="ArmLeft"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.X)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.84" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.08" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.2"  Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.32" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.68" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.88" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="ArmLeft"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.84" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.96" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.08" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.2"  Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.32" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="-15"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.68" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.88" Value="-25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- ARM RIGHT -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="ArmRight"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(RotateTransform.Angle)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.66" Value="25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.2"  Value="25"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.4"  Value="35"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="-15"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="ArmRight"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.36" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.42" Value="6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.66" Value="-12.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.2"  Value="-12.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.4"  Value="-17.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="12.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.8"  Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.08" Value="5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:2.4"  Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- LEG 1 -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg1"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="0.6"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="1"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg1"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-17.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="-17.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- LEG 2 -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg2"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="1"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="0.8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="0.8"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="1"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="1"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg2"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- LEG 3 -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg3"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(RotateTransform.Angle)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.12" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="-10"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg3"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[1].(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="2.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.12" Value="2.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="7.5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

      <!-- LEG 4 -->
      <DoubleAnimationUsingKeyFrames Storyboard.TargetName="Leg4"
        Storyboard.TargetProperty="(UIElement.RenderTransform).(TranslateTransform.Y)">
        <LinearDoubleKeyFrame KeyTime="0:0:0"    Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.44" Value="0"/>
        <LinearDoubleKeyFrame KeyTime="0:0:0.75" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.35" Value="-5"/>
        <LinearDoubleKeyFrame KeyTime="0:0:1.48" Value="0"/>
        <DiscreteDoubleKeyFrame KeyTime="0:0:4"  Value="0"/>
      </DoubleAnimationUsingKeyFrames>

    </Storyboard>
  </Window.Resources>

  <Border Background="#212121" CornerRadius="16" ClipToBounds="True">
    <Viewbox Stretch="Uniform">
      <Canvas Width="160" Height="210">
        <Canvas Canvas.Left="-40" Canvas.Top="-15" Width="240" Height="280">

          <Ellipse x:Name="Shadow" Canvas.Left="75" Canvas.Top="206"
                   Width="90" Height="16" Fill="#111111" Opacity="0.6"
                   RenderTransformOrigin="0.5,0.5">
            <Ellipse.RenderTransform>
              <ScaleTransform ScaleX="1" ScaleY="1"/>
            </Ellipse.RenderTransform>
          </Ellipse>

          <Canvas x:Name="CharGroup" Width="240" Height="280">
            <Canvas.RenderTransform>
              <TransformGroup>
                <ScaleTransform ScaleX="1" ScaleY="1" CenterX="120" CenterY="210"/>
                <TranslateTransform X="0" Y="0"/>
              </TransformGroup>
            </Canvas.RenderTransform>

            <Canvas x:Name="BodyTilt" Width="240" Height="280">
              <Canvas.RenderTransform>
                <RotateTransform Angle="0" CenterX="120" CenterY="150"/>
              </Canvas.RenderTransform>

              <Rectangle x:Name="ArmLeft" Canvas.Left="65" Canvas.Top="140"
                         Width="20" Height="20" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TransformGroup>
                    <RotateTransform Angle="0" CenterX="20" CenterY="10"/>
                    <TranslateTransform X="0" Y="0"/>
                  </TransformGroup>
                </Rectangle.RenderTransform>
              </Rectangle>

              <Rectangle x:Name="Leg1" Canvas.Left="85" Canvas.Top="180"
                         Width="10" Height="30" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TransformGroup>
                    <ScaleTransform ScaleX="1" ScaleY="1" CenterX="5" CenterY="0"/>
                    <TranslateTransform X="0" Y="0"/>
                  </TransformGroup>
                </Rectangle.RenderTransform>
              </Rectangle>
              <Rectangle x:Name="Leg2" Canvas.Left="105" Canvas.Top="180"
                         Width="10" Height="30" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TransformGroup>
                    <ScaleTransform ScaleX="1" ScaleY="1" CenterX="5" CenterY="0"/>
                    <TranslateTransform X="0" Y="0"/>
                  </TransformGroup>
                </Rectangle.RenderTransform>
              </Rectangle>
              <Rectangle x:Name="Leg3" Canvas.Left="125" Canvas.Top="180"
                         Width="10" Height="30" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TransformGroup>
                    <RotateTransform Angle="0" CenterX="5" CenterY="0"/>
                    <TranslateTransform X="0" Y="0"/>
                  </TransformGroup>
                </Rectangle.RenderTransform>
              </Rectangle>
              <Rectangle x:Name="Leg4" Canvas.Left="145" Canvas.Top="180"
                         Width="10" Height="30" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TranslateTransform X="0" Y="0"/>
                </Rectangle.RenderTransform>
              </Rectangle>

              <Rectangle Canvas.Left="85" Canvas.Top="120" Width="70" Height="60" Fill="#D87B60"/>

              <Rectangle x:Name="ArmRight" Canvas.Left="155" Canvas.Top="140"
                         Width="20" Height="20" Fill="#D87B60">
                <Rectangle.RenderTransform>
                  <TransformGroup>
                    <RotateTransform Angle="0" CenterX="0" CenterY="10"/>
                    <TranslateTransform X="0" Y="0"/>
                  </TransformGroup>
                </Rectangle.RenderTransform>
              </Rectangle>

              <Canvas x:Name="FaceShift">
                <Canvas.RenderTransform>
                  <TranslateTransform X="0" Y="0"/>
                </Canvas.RenderTransform>
                <Rectangle Canvas.Left="95"  Canvas.Top="135" Width="10" Height="10" Fill="Black"/>
                <Rectangle Canvas.Left="135" Canvas.Top="135" Width="10" Height="10" Fill="Black"/>
              </Canvas>

            </Canvas>
          </Canvas>

        </Canvas>
      </Canvas>
    </Viewbox>
  </Border>
</Window>
"@

$reader = [System.Xml.XmlReader]::Create([System.IO.StringReader]$xaml)
$window = [System.Windows.Markup.XamlReader]::Load($reader)

$window.Add_Loaded({ $window.Resources["Anim"].Begin() })
$window.Add_MouseLeftButtonDown({ $window.Close() })

$area        = [System.Windows.SystemParameters]::WorkArea
$window.Left = $area.Right  - 260 - 20
$window.Top  = $area.Bottom - 290 - 20

$timer          = New-Object System.Windows.Threading.DispatcherTimer
$timer.Interval = [TimeSpan]::FromSeconds(7)
$timer.Add_Tick({ $timer.Stop(); $window.Close() })
$timer.Start()

$window.ShowDialog() | Out-Null
```

---

## Customization

| What | Where |
|------|-------|
| Popup size | `Width` / `Height` on the `<Window>` element |
| Auto-dismiss delay | `FromSeconds(7)` near the bottom of the script |
| Character color | `Fill="#D87B60"` on the Rectangle elements |
| Background color | `Background="#212121"` on the `<Border>` element |
| Corner radius | `CornerRadius="16"` on the `<Border>` element |

---

## How I built this

The character is the Claude Code mascot rendered purely in WPF (Windows Presentation Foundation) — no browser, no external dependencies. The original design is in `notification_1.svg`.

The animation was hand-translated from CSS `@keyframes` into WPF `Storyboard` / `DoubleAnimationUsingKeyFrames`. CSS `cubic-bezier` curves map directly to WPF `SplineDoubleKeyFrame` with a `KeySpline` attribute. The full animation has 10+ tracks: jump arc, squash & stretch, shadow pulse, body tilt, face look, arm wave, arm drag, and four leg animations.

The popup runs as a detached PowerShell process so it doesn't block Claude Code's response cycle.
