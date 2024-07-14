# UdonTask
![](https://img.shields.io/badge/unity-2022.3+-000.svg)
[![Releases](https://img.shields.io/github/release/chiugame/udon-task.svg)](https://github.com/chiugame/udon-task/releases)

This package enables the execution of Udon from a separate thread.

**Machine translated from Japanese to English.**

## Installation
Open [Iwashi Packages](https://vpm.iwa.si/) and click "Add to VCC" to add UdonTask from VCC.

## Usage
### Basic
```csharp
using UnityEngine;
using UdonSharp;
using VRC.Udon.Common.Interfaces;
using Iwashi.UdonTask;

public class UdonTaskSample : UdonSharpBehaviour
{
  public void ExecuteTask()
  {
    UdonTask.New((IUdonEventReceiver)this, nameof(OnProcess), nameof(OnComplete));
  }

  public void OnProcess()
  {
    /* Write heavy processing here.
     * Note that you cannot touch anything that can only be accessed from the main thread. Be careful! */
  }

  public void OnComplete()
  {
    Debug.Log("Completed!");
  }
}
```

### With Return Values and Arguments
```csharp
using UnityEngine;
using UdonSharp;
using VRC.Udon.Common.Interfaces;
using Iwashi.UdonTask;

public class UdonTaskSample : UdonSharpBehaviour
{
	public void ExecuteTask()
	{
		UdonTask.New((IUdonEventReceiver)this, nameof(OnProcess), nameof(OnComplete), "onProcessContainer", "onReturnContainer", "Iwashi");
	}

	public UdonTaskContainer OnProcess(UdonTaskContainer onProcessContainer)
	{
		var container = UdonTaskContainer.New();
		var str = onProcessContainer.GetVariable<string>(0);
		container = container.AddVariable($"{str}-mo");
		Debug.Log(container.GetVariable<string>(container.Count() - 1));
		return container;
	}

	public void OnComplete(UdonTaskContainer onReturnContainer)
	{
		Debug.Log(onReturnContainer.GetVariable<string>(0));
	}
}
```

- Specify `IUdonEventReceiver` as the first argument. You can set your own `UdonSharpBehaviour` by using `(IUdonEventReceiver)this`.
- Processes that take more than 10 seconds will cause Udon to crash, so they cannot be executed. Try to split them into chunks of around 9.9 seconds.
- When using arguments, ensure that function arguments have unique names within the script.
  - You need to specify the argument names when calling `UdonTask.New`.

## Samples
You can import sample scenes from Unity's Package Manager under UdonTask → Samples.

In the sample scenes, you can test loading 17 images encoded in [Base64](https://gist.githubusercontent.com/chiugame/76a08e9e2cb0735b1c7ff848e335b30f/raw/b956b266e4f0c35b8fde9edb284fe7efc300ba05/SamplePictures.txt) at high speed.

[Test World](https://vrchat.com/home/world/wrld_687f009c-fffb-4532-bb55-c075788a33b1)

## TIPS
- Calling `SendCustomEventDelayedSeconds`/`SendCustomEventDelayedFrames` from a separate thread can invoke the main thread.
- If you access fields being touched by a separate thread from the main thread, Udon may crash depending on the timing.
  - It might crash due to timing issues. The UdonVM stack seems to get misaligned? Specific fields might transform into different types.
- You can output `Debug.Log` even from a separate thread.

## Note
- Implemented thread-safe Udon log output using Harmony.
- Implemented a thread-safe pseudo-random generator class.
