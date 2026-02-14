# LWindows --- Complete Cheat-Sheet

## Main Functions

To require it, do: `require"windows.h"`

``` func
ToWindowProc(hWnd, lfnc) -> userdata
```

Converts a Lua function into a Win32 `WNDPROC`. Keeps the closure alive.
[Important: if the garbage collector collects the closure while the
window is still active, the WNDPROC becomes invalid and your application
may crash.]

``` func
Num2Addr(int) -> userdata
```

Converts an integer into `userdata` representing a pointer.

``` func
Addr2Num(userdata) -> int
```

Converts a `userdata` containing a pointer into an integer.

``` func
Val2Addr(val) -> userdata
```

Creates a memory/userdata block from a Lua value:

- Int, Number, Boolean → simple copy
- Nil + size → empty buffer
- Table → recursive binary blob

``` func
Addr2Val(userdata(nativeFuncPtr), returnType, {argTypes}) -> lfnc
```

Converts a native function pointer into a safe Lua closure. LWindows
internally handles up to 16 arguments. [Arguments beyond this limit will
raise an error.]

``` func
Addr2Val(userdata, valueType) -> anyValue
```

Reads memory from a pointer and returns the specified Lua type. Supports
basic types and pointers; any invalid access produces a safe Lua error
instead of crashing.

``` func
LoopMessages() -> boolean
```

Processes the Windows message queue until the application closes.
Returns `true` if it finishes correctly, or `nil` and an error code if
it fails.

[Internally, closures registered as WinProc receive the messages and
must return `DefWindowProc` if unhandled. LWindows automatically routes
messages by `HWND` and class name.]

## Type Aliases

- `i` or `integer`
- `ir` or `integerref`
- `n` or `number`
- `nr` or `numberref`
- `s` or `string`
- `u` or `userdata`
- `b` or `boolean`
- `br` or `booleanref`
- `p` or `lightuserdata`
- `v` or `void`

[\*ref\* types are references, used to pass and update values directly
in memory.]

## Common Constants and Macros

- `MB_OK`, `MB_YESNO` - MessageBox types
- `WS_OVERLAPPEDWINDOW`, `WS_CHILD`,
  `WS_VISIBLE` - Window styles
- `SW_SHOW`, `SW_HIDE` - ShowWindow commands
- `COLOR_WINDOW`, `COLOR_BTNFACE` - Predefined
  colors
- `IDC_ARROW`, `IDC_HAND` - Predefined cursors
- `IDI_APPLICATION`, `IDI_WARNING` - Predefined
  icons
- `CS_HREDRAW`, `CS_VREDRAW` - Window class styles
- `WM_DESTROY`, `WM_COMMAND` - Window messages

## Advanced Tips and Safety

- LWindows uses SEH (`__try/__except`) so memory faults don't crash Lua.
- Keep alive the closures you pass to Win32 while the window exists.
- Value references are managed via `Val2Addr` to pass and update memory
  directly.
- Avoid passing more than 16 arguments to `Addr2Val` for native
  functions.
- Return `DefWindowProc` in callbacks if a message is not handled.
- LWindows routes messages by HWND and class name automatically,
  allowing multiple controls without conflicts.
- You don't need to specify `sizeof` for standard structs (RECT, POINT,
  WINDOWPLACEMENT).

## Quick Example


    require"windows.h"
    local k32 = GetModuleHandle("kernel32.dll")
    local GetTickCount_addr = GetProcAddress(k32, "GetTickCount")
    local gettick = Addr2Val(Num2Addr(GetTickCount_addr), "integer", {})
    print("Ticks:", gettick())

    local nameBuf, bufsz = Val2Addr(nil, 256)
    local sizeRef = Val2Addr(bufsz)
    local getComputerName = Addr2Val(
        Num2Addr(GetProcAddress(k32,"GetComputerNameA")),
        "boolean",
        {"string","lightuserdata"}
    )
    if getComputerName(nameBuf, sizeRef) then
        print("PC:", nameBuf)
    end

## Minimal Window Example in Lua with LWindows


    require "windows.h"

    local function onButtonClick(hwnd)
        MessageBox(hwnd, "Hello world!", "My first window", MB_OK)
    end

    local winProc = function(hWnd, uMsg, wParam, lParam)
        if uMsg == WM_DESTROY then
            PostQuitMessage(0)
            return 0
        elseif uMsg == WM_COMMAND then
            if LOWORD(wParam) == 1001 then
                onButtonClick(hWnd)
            end
        end
        return DefWindowProc(hWnd, uMsg, wParam, lParam)
    end

    local CLASS_NAME = "LuaClass"
    local wc = {
        style = CS_HREDRAW | CS_VREDRAW,
        lpfnWndProc = winProc,
        cbClsExtra = 0,
        cbWndExtra = 0,
        hInstance = GetModuleHandle(nil),
        hIcon = nil,
        hCursor = LoadCursor(nil, IDC_ARROW),
        hbrBackground = COLOR_WINDOW+1,
        lpszMenuName = nil,
        lpszClassName = CLASS_NAME,
        hIconSm = nil
    }
    RegisterClassEx(wc)

    local hwnd = CreateWindow(
        CLASS_NAME,
        "Hello World from Lua",
        WS_OVERLAPPEDWINDOW,
        100, 100, 400, 300,
        nil, nil,
        wc.hInstance
    )

    local hButton = CreateWindow(
        "BUTTON",
        "Click me",
        WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
        50, 50, 100, 30,
        hwnd,
        1001,
        wc.hInstance
    )

    ShowWindow(hwnd, SW_SHOW)
    UpdateWindow(hwnd)
    LoopMessages()

This example creates a simple window with a button. `LoopMessages()`
processes events. Message routing and closure management are handled
internally by LWindows.