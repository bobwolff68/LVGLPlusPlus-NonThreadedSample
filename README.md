# LVGLPlusPlus NON-THREADED Example

The pair of examples (with and without threading) are intended to both educate on how to use some of the LVGLPlusPlus constructs as well as to be used as "starting templates" for those who wish to have a known good setup that functions in these configurations.

The examples currently are functional on Mac in so-called 'native' compilation and utilize SDL2 and lv_drivers to do the screen/graphics platform support. The examples are also functional on an ESP32 DevKit circa 2021 attached to a 2.4" TFT touchscreen display based on the ILI9341 and this utilizes the Bodmer library "TFT_eSPI" (which is excellent). The IO pins for the TFT controls must be gleaned from the platform.ini file. Your connections may be completely different from these.

## Specific non-threaded differences

I have tried to keep the changes between threaded and non-threaded to a bare minimum. The primary, structural difference, is that in the non-threaded version there is no need to protect any calls into the LVGL library. The LVGL library is not threadsafe and you will get crashes and errors if you use it in a multi-threaded environment without the use of mutexes. Ask me how I know. <cough-cough>

- Primary: hal/main_esp32.cpp and hal/main_emulated.cpp each no longer contain a LVTaskHandler instance for the mutex protection around the call to lv_task_handler(). This is handled in LVTaskHandler::Run() in the threaded version.
- Primary: the 'loop' now must call widgets_update() to facilitate the UI updates - this happens alongside lv_task_handler()
- Secondary: Removed "TheBrain.*" entirely and scooted the UI update mechanism into GlobalObjects.cpp
- Secondary: Added a Timer class for handling the slow UI update which was handled by TheBrain in the threaded case.
- Tertiary: Widgets.cpp has a button handler OnClicked() which was calling TheBrain to add time to the timer. In this new case, it simply calls a function in GlobalObjects.cpp to facilitate the same.

Ultimately this example is simpler to understand, but the threaded version (I feel) is more real-world in that UI and device based objects become threaded such that everyone runs in parallel (or at least many do) which makes for a more easily expandable and smooth running system in the long haul.

## To Do Items

- Document the SDL bit on the Mac (homebrew and associated platform.ini items)
- Document use of lv_drivers in LVGLPlusPlus and how it is present in esp32 build
- 