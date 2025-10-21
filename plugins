"""
IDA Window Name Randomizer Plugin
Automatically changes IDA window names to random names on startup. Anti-cheat love searching for ida name.
"""

import ida_kernwin
import idaapi
import ctypes
import random
import string

# Windows API declarations
try:
    user32 = ctypes.windll.user32
    
    SetWindowTextW = user32.SetWindowTextW
    SetWindowTextW.argtypes = [ctypes.c_void_p, ctypes.c_wchar_p]
    SetWindowTextW.restype = ctypes.c_bool
    
    GetWindowTextW = user32.GetWindowTextW
    GetWindowTextLengthW = user32.GetWindowTextLengthW
    
    EnumWindows = user32.EnumWindows
    GetWindowThreadProcessId = user32.GetWindowThreadProcessId
    IsWindowVisible = user32.IsWindowVisible
    GetCurrentProcessId = ctypes.windll.kernel32.GetCurrentProcessId
except:
    pass

# Plugin configuration
PLUGIN_NAME = "IDA Window Randomizer"
PLUGIN_HOTKEY = ""
PLUGIN_COMMENT = "Randomizes IDA window names"
PLUGIN_HELP = "This plugin automatically changes window names containing 'IDA' to random names"
PLUGIN_VERSION = "1.0"

class WindowNameRandomizer:
    """Handles window name randomization using Windows API"""
    
    def __init__(self):
        self.original_title = None
        self.random_name = None
        self.main_window_hwnd = None
        self.current_pid = GetCurrentProcessId()
        
    def generate_random_name(self):
        """Generate a random application name"""
        prefixes = [
            "System", "Application", "Process", "Service", "Manager",
            "Handler", "Controller", "Monitor", "Viewer", "Editor",
            "Tool", "Utility", "Helper", "Agent", "Client"
        ]
        
        suffixes = [
            "Core", "Pro", "Plus", "Lite", "X", "Suite",
            "Studio", "Builder", "Analyzer", "Inspector"
        ]
        
        # Random combination
        if random.choice([True, False]):
            name = random.choice(prefixes)
            if random.choice([True, False]):
                name += " " + random.choice(suffixes)
        else:
            # Generate random alphanumeric name
            length = random.randint(8, 12)
            name = ''.join(random.choices(string.ascii_uppercase, k=1)) + \
                   ''.join(random.choices(string.ascii_lowercase + string.digits, k=length-1))
        
        return name
    
    def get_window_text(self, hwnd):
        """Get the text of a window"""
        try:
            length = GetWindowTextLengthW(hwnd)
            if length == 0:
                return ""
            
            buff = ctypes.create_unicode_buffer(length + 1)
            GetWindowTextW(hwnd, buff, length + 1)
            return buff.value
        except Exception:
            return ""
    
    def find_all_ida_windows(self):
        """Find ALL IDA windows (visible and hidden) by enumerating windows"""
        found_windows = []
        
        def enum_callback(hwnd, lparam):
            try:
                # Get process ID for this window
                pid = ctypes.c_ulong()
                GetWindowThreadProcessId(hwnd, ctypes.byref(pid))
                
                # Check if it belongs to our process
                if pid.value == self.current_pid:
                    # Get window title (check both visible and hidden windows)
                    title = self.get_window_text(hwnd)
                    if title and "ida" in title.lower():
                        found_windows.append((hwnd, title))
            except:
                pass
            return True
        
        try:
            # Create callback function
            EnumWindowsProc = ctypes.WINFUNCTYPE(ctypes.c_bool, ctypes.c_void_p, ctypes.c_void_p)
            callback = EnumWindowsProc(enum_callback)
            
            # Enumerate all windows
            EnumWindows(callback, None)
            
            return found_windows
        except Exception as e:
            print(f"[{PLUGIN_NAME}] Error: {str(e)}")
            return []
    
    def randomize_main_window(self):
        """Randomize ALL IDA window titles"""
        try:
            # Find ALL IDA windows
            ida_windows = self.find_all_ida_windows()
            
            if not ida_windows:
                return False
            
            # Generate random name if not already done
            if not self.random_name:
                self.random_name = self.generate_random_name()
            
            # Randomize all windows
            success_count = 0
            for hwnd, original_title in ida_windows:
                try:
                    # Determine what to set based on the original title
                    if "quick start" in original_title.lower():
                        new_title = self.random_name
                    elif original_title.strip().lower() == "ida":
                        new_title = self.random_name
                    else:
                        # For main window with file name, keep structure but replace "IDA"
                        if " - " in original_title:
                            parts = original_title.split(" - ", 1)
                            new_title = f"{self.random_name} - {parts[1]}"
                        else:
                            new_title = self.random_name
                    
                    # Set new window title
                    if SetWindowTextW(hwnd, new_title):
                        success_count += 1
                        
                except:
                    pass
            
            if success_count > 0:
                print(f"[{PLUGIN_NAME}] Randomized {success_count} window(s) to '{self.random_name}'")
            
            return success_count > 0
            
        except Exception as e:
            print(f"[{PLUGIN_NAME}] Error: {str(e)}")
            return False


class UIHooks(ida_kernwin.UI_Hooks):
    """UI Hooks to detect when IDA is ready"""
    
    def __init__(self, randomizer):
        super(UIHooks, self).__init__()
        self.randomizer = randomizer
        self.done = False
    
    def ready_to_run(self):
        """Called when IDA is ready"""
        if not self.done:
            try:
                self.randomizer.randomize_main_window()
                self.done = True
            except:
                pass


class IDAWindowRandomizerPlugin(idaapi.plugin_t):
    """IDA Plugin class"""
    
    flags = idaapi.PLUGIN_FIX | idaapi.PLUGIN_HIDE
    comment = PLUGIN_COMMENT
    help = PLUGIN_HELP
    wanted_name = PLUGIN_NAME
    wanted_hotkey = PLUGIN_HOTKEY
    
    def __init__(self):
        super(IDAWindowRandomizerPlugin, self).__init__()
        self.randomizer = None
        self.hooks = None
    
    def init(self):
        """Initialize plugin"""
        try:
            # Check if running on Windows
            import os
            if os.name != 'nt':
                return idaapi.PLUGIN_SKIP
            
            # Create randomizer instance
            self.randomizer = WindowNameRandomizer()
            
            # Install UI hooks to catch when IDA is ready
            self.hooks = UIHooks(self.randomizer)
            self.hooks.hook()
            
            return idaapi.PLUGIN_KEEP
        except:
            return idaapi.PLUGIN_SKIP
    
    def run(self, arg):
        """Run plugin - manually randomize all IDA windows"""
        try:
            if self.randomizer:
                result = self.randomizer.randomize_main_window()
                if result:
                    ida_kernwin.info(f"{PLUGIN_NAME}\n\nWindow names randomized!")
                else:
                    ida_kernwin.warning(f"{PLUGIN_NAME}\n\nFailed to randomize windows")
            else:
                ida_kernwin.warning(f"{PLUGIN_NAME}\n\nPlugin not initialized")
        except Exception as e:
            ida_kernwin.warning(f"{PLUGIN_NAME}\n\nError: {str(e)}")
        
        return True
    
    def term(self):
        """Terminate plugin"""
        if self.hooks:
            self.hooks.unhook()


def PLUGIN_ENTRY():
    """Plugin entry point"""
    return IDAWindowRandomizerPlugin()

