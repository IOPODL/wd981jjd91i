#ifndef UNICODE
#define UNICODE
#endif

#ifndef _WIN32_IE
#define _WIN32_IE 0x0600
#endif

#ifndef _WIN32_WINNT
#define _WIN32_WINNT 0x0601
#endif

#ifndef _UNICODE
#define _UNICODE
#endif

#define WIN32_LEAN_AND_MEAN
#define NOMINMAX

#include <windows.h>
#include <commctrl.h>
#include <shlobj.h>
#include <shellapi.h>
#include <shlwapi.h>
#include <winhttp.h>
#include <ole2.h>
#include <shldisp.h>
#include <dwmapi.h>
#include <gdiplus.h>
#include <tlhelp32.h>

#include <string>
#include <vector>
#include <thread>
#include <atomic>
#include <mutex>
#include <regex>
#include <algorithm>
#include <cstdlib>
#include <ctime>
#include <cstring>
#include <cwchar>
#include <cwctype>
#include <cctype>
#include <iterator>
#include <memory>
#include <initializer_list>
#include <new>
#include <climits>

#pragma comment(lib, "user32.lib")
#pragma comment(lib, "gdi32.lib")
#pragma comment(lib, "comctl32.lib")
#pragma comment(lib, "shell32.lib")
#pragma comment(lib, "shlwapi.lib")
#pragma comment(lib, "winhttp.lib")
#pragma comment(lib, "ole32.lib")
#pragma comment(lib, "oleaut32.lib")
#pragma comment(lib, "uuid.lib")
#pragma comment(lib, "dwmapi.lib")
#pragma comment(lib, "gdiplus.lib")
#pragma comment(lib, "advapi32.lib")

using namespace Gdiplus;

#define IDC_TAB_BASE        2000
#define IDC_DOWNLOAD_ALL    2100
#define IDC_POWERSHELL      2104
#define IDC_ENABLE_SERVICES 2105
#define IDC_DELETE_HOLYCHECK 2106
#define IDC_CANCEL_DOWNLOAD 2107
#define IDC_COPY_EVERYTHING_1 2108
#define IDC_COPY_EVERYTHING_2 2109
#define IDC_COPY_POWERSHELL 2110
#define IDC_COPY_SITES      2111
#define IDC_TILE_BASE       3000

#define TAB_COUNT           4

#define WM_APP_PROGRESS     (WM_APP + 2)
#define WM_APP_DONE         (WM_APP + 3)
#define WM_APP_LOG          (WM_APP + 4)
#define WM_APP_ITEM_DONE    (WM_APP + 5)
#define WM_APP_REFRESH      (WM_APP + 6)

#define TIMER_PARTICLES     9001

#ifndef DWMWA_USE_IMMERSIVE_DARK_MODE
#define DWMWA_USE_IMMERSIVE_DARK_MODE 20
#endif

#ifndef WINHTTP_FLAG_SECURE_PROTOCOL_TLS1
#define WINHTTP_FLAG_SECURE_PROTOCOL_TLS1 0x00000080
#endif

#ifndef WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_1
#define WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_1 0x00000200
#endif

#ifndef WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_2
#define WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_2 0x00000800
#endif

#ifndef WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_3
#define WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_3 0x00002000
#endif

struct DownloadItem
{
    std::wstring name;
    std::wstring url;
    std::wstring file;
    int tab;
};

struct ParsedUrl
{
    std::wstring host;
    INTERNET_PORT port = 0;
    std::wstring pathAndQuery;
    bool https = false;
};


enum class DownloadError
{
    None,
    Cancelled,
    InvalidUrl,
    OpenSessionFailed,
    ConnectFailed,
    OpenRequestFailed,
    SendFailed,
    ReceiveFailed,
    HttpStatus,
    HtmlInsteadOfFile,
    HtmlLinkNotFound,
    ReadFailed,
    FileOpenFailed,
    FileWriteFailed,
    MoveFailed,
    SizeMismatch,
    TooManyRedirects
};

struct DownloadResult
{
    bool ok = false;
    DownloadError error = DownloadError::None;
    DWORD httpStatus = 0;
    DWORD win32Error = 0;
    unsigned long long contentLength = 0;
    unsigned long long bytesWritten = 0;
    unsigned long long resumeFrom = 0;
    bool resumed = false;
    bool retryable = false;
    std::wstring finalUrl;
    std::wstring message;
};

static DownloadResult MakeDownloadError(DownloadError error, DWORD win32Error = 0, bool retryable = true)
{
    DownloadResult result;
    result.ok = false;
    result.error = error;
    result.win32Error = win32Error;
    result.retryable = retryable;
    return result;
}

static DownloadResult MakeDownloadOk()
{
    DownloadResult result;
    result.ok = true;
    result.error = DownloadError::None;
    return result;
}

struct Particle
{
    float x;
    float y;
    float vx;
    float vy;
    float size;
    BYTE alpha;
};

HINSTANCE g_hInst = nullptr;
HWND g_hMain = nullptr;
HWND g_hDownloadAll = nullptr;
HWND g_hPowerShell = nullptr;
HWND g_hEnableServices = nullptr;
HWND g_hDeleteHolyCheck = nullptr;
HWND g_hCancelDownload = nullptr;
HWND g_hCopyEverything1 = nullptr;
HWND g_hCopyEverything2 = nullptr;
HWND g_hCopyPowerShell = nullptr;
HWND g_hCopySites = nullptr;

HFONT g_hFont = nullptr;
HFONT g_hSmallFont = nullptr;
HFONT g_hTinyFont = nullptr;

std::vector<HWND> g_tabButtons;
std::vector<HWND> g_tileButtons;
std::vector<Particle> g_particles;

std::atomic_bool g_working(false);
std::atomic_bool g_cancelRequested(false);
std::atomic_bool g_deleteInProgress(false);
std::atomic_bool g_destroying(false);
std::atomic_int g_activeSingleDownloads(0);
std::atomic_int g_singleTotal(0);
std::atomic_int g_singleDone(0);
std::mutex g_zipMutex;
std::mutex g_logMutex;
std::mutex g_activeDownloadsMutex;
std::mutex g_itemBusyMutex;
std::mutex g_httpHandlesMutex;
std::mutex g_ericDownloadMutex;

struct HttpHandleSlot
{
    HINTERNET handle = nullptr;
    bool closed = false;
};

std::vector<std::shared_ptr<HttpHandleSlot>> g_httpHandles;
std::vector<BYTE> g_itemBusy;
std::vector<std::wstring> g_logLines;
std::vector<std::wstring> g_activeDownloads;
int g_logScrollOffset = 0;

int g_currentTab = 0;
int g_progressPercent = 0;
ULONG_PTR g_gdiplusToken = 0;
bool g_particlesPaused = false;
DWORD g_uiThreadId = 0;

static const size_t HTML_READ_LIMIT = 4 * 1024 * 1024;
static const DWORD ZIP_EXTRACT_TIMEOUT_MS = 120000;
static const DWORD ZIP_EXTRACT_POLL_MS = 250;

const COLORREF BG = RGB(18, 18, 19);
const COLORREF PANEL = RGB(40, 40, 44);
const COLORREF PANEL_HOVER = RGB(56, 56, 62);
const COLORREF PANEL_PRESSED = RGB(70, 70, 76);
const COLORREF BORDER = RGB(72, 72, 80);
const COLORREF TEXT = RGB(242, 242, 245);
const COLORREF BLUE_LIGHT = RGB(115, 165, 255);

static const int LOG_PANEL_X = 28;
static const int LOG_PANEL_W = 438;
static const int LOG_PANEL_H = 86;
static const int LOG_PANEL_BOTTOM_MARGIN = 18;
static const int LOG_INVALIDATE_HEIGHT = 126;
static const size_t LOG_MAX_LINES = 200;
static const size_t LOG_VISIBLE_LINES = 4;
static const int CANCEL_BUTTON_W = 84;
static const int CANCEL_BUTTON_H = 27;
static const int HELPER_PANEL_X = 28;
static const int HELPER_PANEL_W = 418;
static const int HELPER_PANEL_H = 146;
static const int COPY_BUTTON_W = 82;
static const int COPY_BUTTON_H = 22;

static RECT GetDownloadLogRect(const RECT& rc)
{
    RECT out{};
    out.left = LOG_PANEL_X;
    out.right = LOG_PANEL_X + LOG_PANEL_W;
    out.top = rc.bottom - LOG_PANEL_H - LOG_PANEL_BOTTOM_MARGIN;
    out.bottom = out.top + LOG_PANEL_H;

    if (out.top < 76)
    {
        out.top = 76;
        out.bottom = out.top + LOG_PANEL_H;
    }

    return out;
}

static RECT GetHelperTextRect(const RECT& rc)
{
    RECT logRect = GetDownloadLogRect(rc);
    RECT out{};
    out.left = HELPER_PANEL_X;
    out.right = HELPER_PANEL_X + HELPER_PANEL_W;
    out.top = logRect.top - HELPER_PANEL_H - 16;

    if (out.top < 382)
        out.top = 382;

    out.bottom = out.top + HELPER_PANEL_H;

    if (out.bottom > logRect.top - 10)
    {
        out.bottom = logRect.top - 10;
        out.top = out.bottom - HELPER_PANEL_H;
    }

    return out;
}

static bool PointInRect(const RECT& rc, POINT pt)
{
    return pt.x >= rc.left && pt.x < rc.right && pt.y >= rc.top && pt.y < rc.bottom;
}

static int GetMaxLogScrollOffsetLocked()
{
    if (g_logLines.size() <= LOG_VISIBLE_LINES)
        return 0;

    return static_cast<int>(g_logLines.size() - LOG_VISIBLE_LINES);
}

static void ClampLogScrollOffsetLocked()
{
    int maxOffset = GetMaxLogScrollOffsetLocked();

    if (g_logScrollOffset < 0)
        g_logScrollOffset = 0;

    if (g_logScrollOffset > maxOffset)
        g_logScrollOffset = maxOffset;
}

static bool ScrollDownloadLog(int wheelDelta)
{
    std::lock_guard<std::mutex> lock(g_logMutex);

    int maxOffset = GetMaxLogScrollOffsetLocked();

    if (maxOffset <= 0)
        return false;

    int oldOffset = g_logScrollOffset;
    int step = std::max(1, std::abs(wheelDelta) / WHEEL_DELTA);

    if (wheelDelta > 0)
        g_logScrollOffset += step;
    else if (wheelDelta < 0)
        g_logScrollOffset -= step;

    ClampLogScrollOffsetLocked();
    return g_logScrollOffset != oldOffset;
}

static std::vector<DownloadItem> g_items =
{
    { L"Everything",           L"https://www.voidtools.com/Everything-1.5.0.1404a.x64.zip", L"Everything-1.5.0.1404a.x64.zip", 0 },
    { L"ShellBag Analyzer",    L"https://privazer.com/ru/shellbag_analyzer_cleaner.exe", L"shellbag_analyzer_cleaner.exe", 0 },
    { L"RecentFilesView",      L"https://www.nirsoft.net/utils/recentfilesview.zip", L"recentfilesview.zip", 0 },
    { L"SystemInformer",       L"https://downloads.sourceforge.net/project/systeminformer/systeminformer-3.2.25011-release-setup.exe", L"systeminformer-3.2.25011-release-setup.exe", 0 },
    { L"LastActivityView",     L"https://www.nirsoft.net/utils/lastactivityview.zip", L"lastactivityview.zip", 0 },
    { L"ExecutedProgramsList", L"https://www.nirsoft.net/utils/executedprogramslist.zip", L"executedprogramslist.zip", 0 },
    { L"RegScanner",           L"https://www.nirsoft.net/utils/regscanner.zip", L"regscanner.zip", 0 },
    { L"USBDriveLog",          L"https://www.nirsoft.net/utils/usbdrivelog.zip", L"usbdrivelog.zip", 0 },
    { L"VersionChecker",       L"https://github.com/HolyWorldWEB/VersionChecker/releases/download/v1.0.0/MinecraftVersionChecker.exe", L"MinecraftVersionChecker.exe", 0 },
    { L"ProcessHacker",        L"https://github.com/processhacker/processhacker/releases/download/v2.39/processhacker-2.39-setup.exe", L"processhacker-2.39-setup.exe", 0 },
    { L"USBDeview",            L"https://www.nirsoft.net/utils/usbdeview-x64.zip", L"usbdeview-x64.zip", 0 },
    { L"BrowserDownloadsView", L"https://www.nirsoft.net/utils/browserdownloadsview.zip", L"browserdownloadsview.zip", 0 },
    { L"JournalTrace",         L"https://github.com/spokwn/JournalTrace/releases/download/1.2/JournalTrace.exe", L"JournalTrace.exe", 0 },
    { L"WinRAR",               L"https://www.win-rar.com/postdownload.html?&L=4", L"winrar-x64.exe", 0 },

    { L"BAMParser",            L"https://github.com/spokwn/BAM-parser/releases/download/v1.2.8/BAMParser.exe", L"BAMParser.exe", 1 },
    { L"RegistryExplorer",     L"https://github.com/wearest/holycheker/releases/download/wearest/RegistryExplorer.zip", L"RegistryExplorer.zip", 1 },
    { L"JumpListExplorer",     L"https://github.com/wearest/holycheker/releases/download/wearest/JumpListExplorer.zip", L"JumpListExplorer.zip", 1 },
    { L"SimpleUnlocker",       L"https://github.com/wearest/holycheker/releases/download/wearest/simpleunlocker_release.zip", L"simpleunlocker_release.zip", 1 },
    { L"WinPrefetchView",      L"https://www.nirsoft.net/utils/winprefetchview.zip", L"winprefetchview.zip", 1 },
    { L"Recuva",               L"https://www.softportal.com/getsoft-5667-recuva-3.html", L"rcsetup-en.exe", 1 },

    { L"PathsParser",          L"https://github.com/spokwn/PathsParser/releases/download/v1.1/PathsParser.exe", L"PathsParser.exe", 2 },
    { L"Detect It Easy",       L"https://github.com/horsicq/Detect-It-Easy/releases/download/Beta/die_win64_portable_3.10_x64.zip", L"die_win64_portable_3.10_x64.zip", 2 },
    { L"Recaf CLI",            L"https://github.com/Col-E/Recaf-Launcher/releases/download/0.8.8/recaf-cli-0.8.8.jar", L"recaf-cli-0.8.8.jar", 2 },

    { L"BrowsingHistoryView",  L"https://www.nirsoft.net/utils/browsinghistoryview.zip", L"browsinghistoryview.zip", 3 },
    { L"AlternateStreamView",  L"https://www.nirsoft.net/utils/alternatestreamview-x64.zip", L"alternatestreamview-x64.zip", 3 },
    { L"WinDefLogView",        L"https://www.nirsoft.net/utils/windeflogview.zip", L"windeflogview.zip", 3 },
    { L"VolatilityWorkbench",  L"https://www.osforensics.com/downloads/VolatilityWorkbench.zip", L"VolatilityWorkbench.zip", 3 },
    { L"KernelLiveDumpTool",   L"https://github.com/spokwn/KernelLiveDumpTool/releases/download/v1.1/KernelLiveDumpTool.exe", L"KernelLiveDumpTool.exe", 3 }
};

static void PostProgress(int value)
{
    HWND hwnd = g_hMain;

    if (!hwnd || g_destroying.load())
        return;

    PostMessageW(hwnd, WM_APP_PROGRESS, static_cast<WPARAM>(value), 0);
}

static std::wstring ShortLogLine(std::wstring line)
{
    const size_t maxLen = 124;

    for (wchar_t& ch : line)
    {
        if (ch == L'\r' || ch == L'\n' || ch == L'\t')
            ch = L' ';
    }

    if (line.size() > maxLen)
        line = line.substr(0, maxLen - 3) + L"...";

    return line;
}

static void PushLogLine(const std::wstring& line)
{
    std::lock_guard<std::mutex> lock(g_logMutex);

    bool wasAtBottom = g_logScrollOffset == 0;

    g_logLines.push_back(ShortLogLine(line));

    while (g_logLines.size() > LOG_MAX_LINES)
    {
        g_logLines.erase(g_logLines.begin());

        if (g_logScrollOffset > 0)
            --g_logScrollOffset;
    }

    if (!wasAtBottom)
        ++g_logScrollOffset;

    ClampLogScrollOffsetLocked();
}

static void ClearDownloadLog()
{
    std::lock_guard<std::mutex> lock(g_logMutex);
    g_logLines.clear();
    g_logScrollOffset = 0;
}

static void PostLog(const std::wstring& line)
{
    HWND hwnd = g_hMain;

    if (!hwnd || g_destroying.load())
        return;

    std::wstring* heapLine = new (std::nothrow) std::wstring(line);

    if (!heapLine)
        return;

    if (!PostMessageW(hwnd, WM_APP_LOG, 0, reinterpret_cast<LPARAM>(heapLine)))
        delete heapLine;
}

static std::shared_ptr<HttpHandleSlot> TrackHttpHandle(HINTERNET handle)
{
    if (!handle)
        return nullptr;

    auto slot = std::make_shared<HttpHandleSlot>();
    slot->handle = handle;

    std::lock_guard<std::mutex> lock(g_httpHandlesMutex);
    g_httpHandles.push_back(slot);
    return slot;
}

static void CloseTrackedHttpHandle(std::shared_ptr<HttpHandleSlot>& slot)
{
    HINTERNET handle = nullptr;

    {
        std::lock_guard<std::mutex> lock(g_httpHandlesMutex);

        if (!slot || slot->closed)
            return;

        handle = slot->handle;
        slot->handle = nullptr;
        slot->closed = true;

        g_httpHandles.erase(
            std::remove(g_httpHandles.begin(), g_httpHandles.end(), slot),
            g_httpHandles.end()
        );
    }

    if (handle)
        WinHttpCloseHandle(handle);
}

static void AbortActiveHttpHandles()
{
    std::vector<std::shared_ptr<HttpHandleSlot>> handles;

    {
        std::lock_guard<std::mutex> lock(g_httpHandlesMutex);
        handles = g_httpHandles;
    }

    for (auto& slot : handles)
        CloseTrackedHttpHandle(slot);
}

static bool IsCancelRequested()
{
    return g_cancelRequested.load();
}

static void RequestCancelDownloads()
{
    bool hasWork = g_working.load() || g_activeSingleDownloads.load() > 0;

    if (!hasWork)
        return;

    g_cancelRequested = true;
    AbortActiveHttpHandles();
    PostLog(L"Отмена загрузок...");
}


static bool IsItemBusyIndex(int index)
{
    std::lock_guard<std::mutex> lock(g_itemBusyMutex);

    if (index < 0 || index >= static_cast<int>(g_itemBusy.size()))
        return false;

    return g_itemBusy[index] != 0;
}

static void SetItemBusyIndex(int index, bool busy)
{
    std::lock_guard<std::mutex> lock(g_itemBusyMutex);

    if (index < 0 || index >= static_cast<int>(g_itemBusy.size()))
        return;

    g_itemBusy[index] = busy ? 1 : 0;
}

static void SetTileEnabledByIndex(int index, bool enabled)
{
    if (index < 0 || index >= static_cast<int>(g_tileButtons.size()))
        return;

    EnableWindow(g_tileButtons[index], enabled ? TRUE : FALSE);
    InvalidateRect(g_tileButtons[index], nullptr, FALSE);
}

static bool IsUiThread()
{
    return g_uiThreadId != 0 && GetCurrentThreadId() == g_uiThreadId;
}

static void PostUiRefresh()
{
    HWND hwnd = g_hMain;

    if (!hwnd || g_destroying.load())
        return;

    PostMessageW(hwnd, WM_APP_REFRESH, 0, 0);
}

static void InvalidateBottomPanel()
{
    HWND hwnd = g_hMain;

    if (!hwnd || g_destroying.load())
        return;

    if (!IsUiThread())
    {
        PostUiRefresh();
        return;
    }

    RECT rc{};
    GetClientRect(hwnd, &rc);
    RECT panelRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };
    InvalidateRect(hwnd, &panelRect, FALSE);
}

static void ClearActiveDownloads()
{
    std::lock_guard<std::mutex> lock(g_activeDownloadsMutex);
    g_activeDownloads.clear();
}

static void AddActiveDownload(const std::wstring& name)
{
    {
        std::lock_guard<std::mutex> lock(g_activeDownloadsMutex);

        if (std::find(g_activeDownloads.begin(), g_activeDownloads.end(), name) == g_activeDownloads.end())
            g_activeDownloads.push_back(name);
    }

    InvalidateBottomPanel();
}

static void RemoveActiveDownload(const std::wstring& name)
{
    {
        std::lock_guard<std::mutex> lock(g_activeDownloadsMutex);
        g_activeDownloads.erase(std::remove(g_activeDownloads.begin(), g_activeDownloads.end(), name), g_activeDownloads.end());
    }

    InvalidateBottomPanel();
}

static std::wstring GetActiveDownloadsText()
{
    std::lock_guard<std::mutex> lock(g_activeDownloadsMutex);

    if (g_activeDownloads.empty())
        return L"";

    std::wstring result = L"Сейчас: ";

    const size_t maxShown = 3;

    for (size_t i = 0; i < g_activeDownloads.size() && i < maxShown; ++i)
    {
        if (i > 0)
            result += L", ";

        result += g_activeDownloads[i];
    }

    if (g_activeDownloads.size() > maxShown)
        result += L" +" + std::to_wstring(g_activeDownloads.size() - maxShown);

    return result;
}

static std::wstring ToLower(std::wstring s)
{
    std::transform(s.begin(), s.end(), s.begin(), [](wchar_t c)
    {
        return static_cast<wchar_t>(towlower(c));
    });
    return s;
}

static bool EndsWithI(const std::wstring& s, const std::wstring& suffix)
{
    std::wstring a = ToLower(s);
    std::wstring b = ToLower(suffix);

    return a.size() >= b.size() &&
        a.compare(a.size() - b.size(), b.size(), b) == 0;
}

static bool IsFastFailName(const std::wstring& name)
{
    std::wstring lower = ToLower(name);

    return lower.find(L"registryexplorer") != std::wstring::npos ||
        (lower.find(L"registry") != std::wstring::npos && lower.find(L"explorer") != std::wstring::npos) ||
        lower.find(L"jumplistexplorer") != std::wstring::npos ||
        (lower.find(L"jumplist") != std::wstring::npos && lower.find(L"explorer") != std::wstring::npos) ||
        lower.find(L"simpleunlocker") != std::wstring::npos ||
        (lower.find(L"simple") != std::wstring::npos && lower.find(L"unlocker") != std::wstring::npos);
}

static bool IsFastFailItem(const DownloadItem& item)
{
    return IsFastFailName(item.name);
}

static bool IsEricZimmermanUrl(const std::wstring& url)
{
    return ToLower(url).find(L"download.ericzimmermanstools.com") != std::wstring::npos;
}

static bool IsFastFailUrl(const std::wstring& url)
{
    std::wstring lower = ToLower(url);

    // Eric Zimmerman archives are direct but can be large, so they must NOT use
    // the short fast-fail timeout. Browser downloads work because they wait
    // longer; the program should do the same.
    return lower.find(L"simpleunlocker.ds1nc.ru") != std::wstring::npos;
}

static bool IsFastFailDownload(const std::wstring& displayName, const std::wstring& url)
{
    return IsFastFailName(displayName) || IsFastFailUrl(url);
}

static bool IsRegistryExplorerItem(const DownloadItem& item)
{
    std::wstring name = ToLower(item.name);
    return name.find(L"registryexplorer") != std::wstring::npos ||
        (name.find(L"registry") != std::wstring::npos && name.find(L"explorer") != std::wstring::npos);
}

static bool IsJumpListExplorerItem(const DownloadItem& item)
{
    std::wstring name = ToLower(item.name);
    return name.find(L"jumplistexplorer") != std::wstring::npos ||
        (name.find(L"jumplist") != std::wstring::npos && name.find(L"explorer") != std::wstring::npos);
}

static bool IsSimpleUnlockerItem(const DownloadItem& item)
{
    std::wstring name = ToLower(item.name);
    return name.find(L"simpleunlocker") != std::wstring::npos ||
        (name.find(L"simple") != std::wstring::npos && name.find(L"unlocker") != std::wstring::npos);
}

static bool IsWinRarItem(const DownloadItem& item)
{
    return ToLower(item.name).find(L"winrar") != std::wstring::npos;
}

static bool IsSystemInformerItem(const DownloadItem& item)
{
    std::wstring name = ToLower(item.name);
    return name.find(L"systeminformer") != std::wstring::npos ||
        (name.find(L"system") != std::wstring::npos && name.find(L"informer") != std::wstring::npos);
}

static bool IsEricZimmermanLargeItem(const DownloadItem& item)
{
    return IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item);
}

static DWORD GetPowerShellTimeoutMs(const DownloadItem& item)
{
    if (IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item))
        return 180000;

    if (IsSimpleUnlockerItem(item))
        return 60000;

    if (IsWinRarItem(item) || IsSystemInformerItem(item))
        return 45000;

    return IsFastFailItem(item) ? 22000 : 42000;
}

static void GetHttpTimeouts(const std::wstring& displayName, const std::wstring& url, int& resolveMs, int& connectMs, int& sendMs, int& receiveMs)
{
    std::wstring lowerUrl = ToLower(url);

    if (IsEricZimmermanUrl(url))
    {
        // Eric Zimmerman GUI tools are large archives. Short receive timeouts
        // made RegistryExplorer/JumpListExplorer fail or look stuck while the
        // same link worked in a browser. Give the CDN enough time to stream.
        resolveMs = 10000;
        connectMs = 15000;
        sendMs = 15000;
        receiveMs = 600000;
    }
    else if (lowerUrl.find(L"mods.holyworld.me") != std::wstring::npos)
    {
        resolveMs = 3500;
        connectMs = 3500;
        sendMs = 5000;

        if (lowerUrl.find(L"download") != std::wstring::npos ||
            lowerUrl.find(L".zip") != std::wstring::npos ||
            lowerUrl.find(L".exe") != std::wstring::npos ||
            lowerUrl.find(L"file=") != std::wstring::npos ||
            lowerUrl.find(L"app=") != std::wstring::npos ||
            lowerUrl.find(L"name=") != std::wstring::npos)
        {
            receiveMs = 60000;
        }
        else
        {
            receiveMs = 12000;
        }
    }
    else if (IsFastFailDownload(displayName, url))
    {
        resolveMs = 3500;
        connectMs = 3500;
        sendMs = 5000;
        receiveMs = 16000;
    }
    else
    {
        resolveMs = 4500;
        connectMs = 4500;
        sendMs = 8000;
        receiveMs = 26000;
    }
}

static bool FileExists(const std::wstring& path)
{
    DWORD attr = GetFileAttributesW(path.c_str());
    return attr != INVALID_FILE_ATTRIBUTES && !(attr & FILE_ATTRIBUTE_DIRECTORY);
}


static bool GetFileSizeBytes(const std::wstring& path, unsigned long long& outSize)
{
    outSize = 0;

    HANDLE file = CreateFileW(
        path.c_str(),
        GENERIC_READ,
        FILE_SHARE_READ | FILE_SHARE_WRITE,
        nullptr,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
        return false;

    LARGE_INTEGER size{};
    bool ok = GetFileSizeEx(file, &size) != FALSE && size.QuadPart >= 0;
    CloseHandle(file);

    if (!ok)
        return false;

    outSize = static_cast<unsigned long long>(size.QuadPart);
    return true;
}

static unsigned long long GetFileSizeBytesOrZero(const std::wstring& path)
{
    unsigned long long size = 0;
    GetFileSizeBytes(path, size);
    return size;
}

static bool ParseUnsignedLongLong(const std::wstring& value, unsigned long long& out)
{
    out = 0;
    if (value.empty())
        return false;

    size_t i = 0;
    while (i < value.size() && iswspace(value[i]))
        ++i;

    bool hasDigit = false;
    for (; i < value.size(); ++i)
    {
        wchar_t ch = value[i];
        if (iswspace(ch))
            break;

        if (ch < L'0' || ch > L'9')
            return false;

        hasDigit = true;
        unsigned long long digit = static_cast<unsigned long long>(ch - L'0');
        if (out > (ULLONG_MAX - digit) / 10)
            return false;

        out = out * 10 + digit;
    }

    return hasDigit;
}

static void LogDownloadAttemptFailure(const std::wstring& displayName, int attempt, int totalAttempts, const DownloadResult& result)
{
    (void)displayName;
    (void)attempt;
    (void)totalAttempts;
    (void)result;

    // Подробные причины ошибок намеренно не выводятся в UI.
    // Итоговый короткий статус уже пишет ProcessItem(): "Не скачалось" / "Отменено".
}

static DWORD GetRetryDelayMs(int attempt)
{
    static const DWORD delays[] = { 0, 1000, 3000, 7000 };
    if (attempt <= 0)
        return 0;

    size_t index = static_cast<size_t>(attempt);
    if (index >= ARRAYSIZE(delays))
        index = ARRAYSIZE(delays) - 1;

    return delays[index];
}

static bool SleepRetryDelay(DWORD delayMs)
{
    DWORD slept = 0;
    while (slept < delayMs)
    {
        if (IsCancelRequested())
            return false;

        DWORD chunk = std::min<DWORD>(150, delayMs - slept);
        Sleep(chunk);
        slept += chunk;
    }

    return true;
}

static bool DirectoryExists(const std::wstring& path)
{
    DWORD attr = GetFileAttributesW(path.c_str());
    return attr != INVALID_FILE_ATTRIBUTES && (attr & FILE_ATTRIBUTE_DIRECTORY);
}

static bool EnsureDirectory(const std::wstring& path)
{
    if (DirectoryExists(path))
        return true;

    int r = SHCreateDirectoryExW(nullptr, path.c_str(), nullptr);
    return r == ERROR_SUCCESS || r == ERROR_ALREADY_EXISTS || DirectoryExists(path);
}

static std::wstring CombinePath(const std::wstring& a, const std::wstring& b)
{
    if (a.empty())
        return b;

    if (b.empty())
        return a;

    if (b.size() >= 2 && b[1] == L':')
        return b;

    if (b[0] == L'\\' || b[0] == L'/')
        return b;

    std::wstring out = a;

    if (!out.empty() && out.back() != L'\\' && out.back() != L'/')
        out.push_back(L'\\');

    out += b;
    return out;
}

static std::wstring RemoveExtension(std::wstring s)
{
    size_t p = s.find_last_of(L'.');
    if (p != std::wstring::npos)
        s = s.substr(0, p);
    return s;
}

static std::wstring GetTabFolderName(int tab)
{
    switch (tab)
    {
    case 0: return L"Мл.сотрудник";
    case 1: return L"Сотрудник";
    case 2: return L"Вед.сотрудник";
    case 3: return L"Доп.программы";
    default: return L"Прочее";
    }
}

static std::wstring BuildHolyCheckFolderPath(bool ensureDirectory)
{
    PWSTR downloads = nullptr;
    std::wstring result;

    if (SUCCEEDED(SHGetKnownFolderPath(FOLDERID_Downloads, 0, nullptr, &downloads)) && downloads)
    {
        result = CombinePath(downloads, L"holycheck");
        CoTaskMemFree(downloads);
    }
    else
    {
        wchar_t profile[MAX_PATH]{};
        GetEnvironmentVariableW(L"USERPROFILE", profile, MAX_PATH);
        result = CombinePath(CombinePath(profile, L"Downloads"), L"holycheck");
    }

    if (ensureDirectory)
        EnsureDirectory(result);

    return result;
}

static std::wstring GetHolyCheckFolder()
{
    return BuildHolyCheckFolderPath(true);
}

static std::wstring GetHolyCheckFolderPathOnly()
{
    return BuildHolyCheckFolderPath(false);
}
static std::wstring GetFullPathSafe(const std::wstring& path)
{
    if (path.empty())
        return L"";

    DWORD required = GetFullPathNameW(path.c_str(), 0, nullptr, nullptr);

    if (required == 0)
        return path;

    std::wstring out(required, L'\0');
    DWORD written = GetFullPathNameW(path.c_str(), required, &out[0], nullptr);

    if (written == 0 || written >= required)
        return path;

    out.resize(written);

    while (!out.empty() && (out.back() == L'\\' || out.back() == L'/'))
        out.pop_back();

    return out;
}

static bool IsSafeHolyCheckDeletionTarget(const std::wstring& folder)
{
    std::wstring full = ToLower(GetFullPathSafe(folder));
    std::wstring expected = ToLower(GetFullPathSafe(GetHolyCheckFolderPathOnly()));

    if (full.empty() || expected.empty())
        return false;

    if (full != expected)
        return false;

    if (full.size() < 12)
        return false;

    return EndsWithI(full, L"\\holycheck") || EndsWithI(full, L"/holycheck");
}

static std::wstring GetMinecraftFolder()
{
    wchar_t appData[MAX_PATH * 4]{};
    DWORD len = GetEnvironmentVariableW(L"APPDATA", appData, ARRAYSIZE(appData));

    std::wstring result;

    if (len > 0 && len < ARRAYSIZE(appData))
    {
        result = CombinePath(appData, L".minecraft");
    }
    else
    {
        wchar_t profile[MAX_PATH * 4]{};
        DWORD profileLen = GetEnvironmentVariableW(L"USERPROFILE", profile, ARRAYSIZE(profile));

        if (profileLen > 0 && profileLen < ARRAYSIZE(profile))
            result = CombinePath(CombinePath(profile, L"AppData\\Roaming"), L".minecraft");
        else
            result = L".minecraft";
    }

    EnsureDirectory(result);
    return result;
}

static std::wstring Utf8ToWide(const std::string& s)
{
    if (s.empty())
        return L"";

    int n = MultiByteToWideChar(CP_UTF8, 0, s.c_str(), static_cast<int>(s.size()), nullptr, 0);
    std::wstring out(n, L'\0');

    if (n > 0)
        MultiByteToWideChar(CP_UTF8, 0, s.c_str(), static_cast<int>(s.size()), &out[0], n);

    return out;
}

static std::string WideToUtf8(const std::wstring& s)
{
    if (s.empty())
        return "";

    int n = WideCharToMultiByte(CP_UTF8, 0, s.c_str(), static_cast<int>(s.size()), nullptr, 0, nullptr, nullptr);
    std::string out(n, '\0');

    if (n > 0)
        WideCharToMultiByte(CP_UTF8, 0, s.c_str(), static_cast<int>(s.size()), &out[0], n, nullptr, nullptr);

    return out;
}

static std::wstring UrlEncodeUtf8(const std::wstring& s)
{
    static const wchar_t* hex = L"0123456789ABCDEF";
    std::string utf8 = WideToUtf8(s);
    std::wstring out;

    for (unsigned char c : utf8)
    {
        if ((c >= 'A' && c <= 'Z') ||
            (c >= 'a' && c <= 'z') ||
            (c >= '0' && c <= '9') ||
            c == '-' || c == '_' || c == '.' || c == '~')
        {
            out.push_back(static_cast<wchar_t>(c));
        }
        else if (c == ' ')
        {
            out += L"%20";
        }
        else
        {
            out.push_back(L'%');
            out.push_back(hex[(c >> 4) & 0x0F]);
            out.push_back(hex[c & 0x0F]);
        }
    }

    return out;
}

static bool ParseUrl(const std::wstring& url, ParsedUrl& out)
{
    URL_COMPONENTS uc{};
    uc.dwStructSize = sizeof(uc);

    wchar_t host[512]{};
    wchar_t path[4096]{};
    wchar_t extra[4096]{};

    uc.lpszHostName = host;
    uc.dwHostNameLength = ARRAYSIZE(host);
    uc.lpszUrlPath = path;
    uc.dwUrlPathLength = ARRAYSIZE(path);
    uc.lpszExtraInfo = extra;
    uc.dwExtraInfoLength = ARRAYSIZE(extra);

    if (!WinHttpCrackUrl(url.c_str(), 0, 0, &uc))
        return false;

    out.host.assign(host, uc.dwHostNameLength);
    out.port = uc.nPort;
    out.https = uc.nScheme == INTERNET_SCHEME_HTTPS;
    out.pathAndQuery.assign(path, uc.dwUrlPathLength);
    out.pathAndQuery.append(extra, uc.dwExtraInfoLength);

    if (out.pathAndQuery.empty())
        out.pathAndQuery = L"/";

    return !out.host.empty();
}

static std::wstring MakeAbsoluteUrl(const std::wstring& base, const std::wstring& href)
{
    if (href.rfind(L"http://", 0) == 0 || href.rfind(L"https://", 0) == 0)
        return href;

    ParsedUrl p;
    if (!ParseUrl(base, p))
        return href;

    std::wstring origin = (p.https ? L"https://" : L"http://") + p.host;

    if (href.rfind(L"//", 0) == 0)
        return p.https ? L"https:" + href : L"http:" + href;

    if (!href.empty() && href[0] == L'/')
        return origin + href;

    std::wstring dir = p.pathAndQuery;
    size_t q = dir.find(L'?');
    if (q != std::wstring::npos)
        dir = dir.substr(0, q);

    size_t slash = dir.find_last_of(L'/');
    dir = slash != std::wstring::npos ? dir.substr(0, slash + 1) : L"/";

    return origin + dir + href;
}

static void ReplaceAllAscii(std::string& s, const std::string& from, const std::string& to)
{
    if (from.empty())
        return;

    size_t pos = 0;
    while ((pos = s.find(from, pos)) != std::string::npos)
    {
        s.replace(pos, from.size(), to);
        pos += to.size();
    }
}

static std::wstring CleanHtmlUrl(std::string raw)
{
    ReplaceAllAscii(raw, "&amp;", "&");
    ReplaceAllAscii(raw, "&quot;", "\"");
    ReplaceAllAscii(raw, "&#34;", "\"");
    ReplaceAllAscii(raw, "&#x22;", "\"");
    ReplaceAllAscii(raw, "&#39;", "'");
    ReplaceAllAscii(raw, "&#x27;", "'");
    ReplaceAllAscii(raw, "&#x2F;", "/");
    ReplaceAllAscii(raw, "\\/", "/");
    ReplaceAllAscii(raw, "\\u002F", "/");
    ReplaceAllAscii(raw, "\\u002f", "/");
    ReplaceAllAscii(raw, "\\u003A", ":");
    ReplaceAllAscii(raw, "\\u003a", ":");
    ReplaceAllAscii(raw, "\\u0026", "&");
    ReplaceAllAscii(raw, "\\u003D", "=");
    ReplaceAllAscii(raw, "\\u003d", "=");
    ReplaceAllAscii(raw, "\\u003F", "?");
    ReplaceAllAscii(raw, "\\u003f", "?");

    while (!raw.empty() && (raw.back() == ',' || raw.back() == ';' || raw.back() == ')' || raw.back() == ']'))
        raw.pop_back();

    return Utf8ToWide(raw);
}

static std::wstring FindBinaryLink(const std::wstring& baseUrl, const std::vector<unsigned char>& bytes)
{
    std::string html(bytes.begin(), bytes.end());

    std::regex hrefRe(
        R"((?:href|src|data-url|data-download-url)\s*=\s*["']([^"']+\.(?:zip|exe|jar|7z|rar|msi)(?:/download)?(?:\?[^"']*)?)["'])",
        std::regex_constants::icase
    );

    std::smatch m;
    auto b = html.cbegin();
    auto e = html.cend();

    while (std::regex_search(b, e, m, hrefRe))
        return MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(m[1].str()));

    std::regex plainRe(
        R"((https?://[^\s"'<>]+?\.(?:zip|exe|jar|7z|rar|msi)(?:/download)?(?:\?[^\s"'<>]+)?))",
        std::regex_constants::icase
    );

    if (std::regex_search(html.cbegin(), html.cend(), m, plainRe))
        return MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(m[1].str()));

    std::regex refreshRe(
        R"((?:url|URL)\s*=\s*([^"'>\s]+?\.(?:zip|exe|jar|7z|rar|msi)(?:/download)?(?:\?[^"'>\s]+)?))",
        std::regex_constants::icase
    );

    if (std::regex_search(html.cbegin(), html.cend(), m, refreshRe))
        return MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(m[1].str()));

    // SoftPortal can give a direct /get-... page or a product /software-... page first.
    // Follow it, then DownloadRecursive will either find the final binary link or fail cleanly.
    std::regex softPortalRe(
        R"((?:href|src)\s*=\s*["']([^"']*(?:getsoft-\d+[^"']*\.html|get-\d+[^"']*\.html|software-\d+[^"']*\.html|/en/[^"']+/download|/en/[^"']+/getsoft)[^"']*)["'])",
        std::regex_constants::icase
    );

    if (std::regex_search(html.cbegin(), html.cend(), m, softPortalRe))
        return MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(m[1].str()));

    std::regex jsLocationRe(
        R"((?:location\.href|window\.location|document\.location)\s*=\s*["']([^"']+)["'])",
        std::regex_constants::icase
    );

    if (std::regex_search(html.cbegin(), html.cend(), m, jsLocationRe))
        return MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(m[1].str()));

    return L"";
}

static DWORD QueryStatus(HINTERNET request)
{
    DWORD status = 0;
    DWORD size = sizeof(status);

    WinHttpQueryHeaders(
        request,
        WINHTTP_QUERY_STATUS_CODE | WINHTTP_QUERY_FLAG_NUMBER,
        WINHTTP_HEADER_NAME_BY_INDEX,
        &status,
        &size,
        WINHTTP_NO_HEADER_INDEX
    );

    return status;
}

static std::wstring QueryHeader(HINTERNET request, DWORD query)
{
    DWORD size = 0;

    WinHttpQueryHeaders(
        request,
        query,
        WINHTTP_HEADER_NAME_BY_INDEX,
        nullptr,
        &size,
        WINHTTP_NO_HEADER_INDEX
    );

    if (GetLastError() != ERROR_INSUFFICIENT_BUFFER || size == 0)
        return L"";

    std::wstring value(size / sizeof(wchar_t), L'\0');

    if (!WinHttpQueryHeaders(
        request,
        query,
        WINHTTP_HEADER_NAME_BY_INDEX,
        &value[0],
        &size,
        WINHTTP_NO_HEADER_INDEX))
    {
        return L"";
    }

    while (!value.empty() && value.back() == L'\0')
        value.pop_back();

    return value;
}

static unsigned long long QueryContentLength(HINTERNET request)
{
    std::wstring header = QueryHeader(request, WINHTTP_QUERY_CONTENT_LENGTH);
    unsigned long long value = 0;
    return ParseUnsignedLongLong(header, value) ? value : 0;
}

static bool ReadAll(HINTERNET request, std::vector<unsigned char>& out, size_t maxBytes = HTML_READ_LIMIT)
{
    out.clear();

    for (;;)
    {
        if (IsCancelRequested())
            return false;

        DWORD available = 0;

        if (!WinHttpQueryDataAvailable(request, &available))
            return false;

        if (available == 0)
            break;

        if (out.size() + static_cast<size_t>(available) > maxBytes)
            return false;

        size_t oldSize = out.size();
        out.resize(oldSize + available);

        DWORD read = 0;

        if (IsCancelRequested())
            return false;

        if (!WinHttpReadData(request, out.data() + oldSize, available, &read))
            return false;

        out.resize(oldSize + read);

        if (read == 0)
            break;
    }

    return true;
}

static bool LooksLikeHtml(const std::wstring& contentType)
{
    std::wstring ct = ToLower(contentType);
    return ct.find(L"text/html") != std::wstring::npos ||
        ct.find(L"application/xhtml") != std::wstring::npos;
}

static bool IsRetryableHttpStatus(DWORD status)
{
    return status == 408 || status == 416 || status == 429 || status >= 500;
}

static DownloadResult DownloadRecursiveOnce(
    const std::wstring& url,
    const std::wstring& targetPath,
    const std::wstring& displayName,
    int depth)
{
    if (IsCancelRequested())
        return MakeDownloadError(DownloadError::Cancelled, 0, false);

    if (depth > 4)
        return MakeDownloadError(DownloadError::TooManyRedirects, 0, false);

    ParsedUrl p;

    if (!ParseUrl(url, p))
        return MakeDownloadError(DownloadError::InvalidUrl, GetLastError(), false);

    HINTERNET session = nullptr;
    HINTERNET connect = nullptr;
    HINTERNET request = nullptr;
    std::shared_ptr<HttpHandleSlot> sessionSlot;
    std::shared_ptr<HttpHandleSlot> connectSlot;
    std::shared_ptr<HttpHandleSlot> requestSlot;

    auto finish = [&](DownloadResult result) -> DownloadResult
    {
        CloseTrackedHttpHandle(requestSlot);
        CloseTrackedHttpHandle(connectSlot);
        CloseTrackedHttpHandle(sessionSlot);
        return result;
    };

    session = WinHttpOpen(
        L"Mozilla/5.0 HolyCheckDownloader/9.3",
        WINHTTP_ACCESS_TYPE_DEFAULT_PROXY,
        WINHTTP_NO_PROXY_NAME,
        WINHTTP_NO_PROXY_BYPASS,
        0
    );

    if (!session)
        return MakeDownloadError(DownloadError::OpenSessionFailed, GetLastError(), true);

    sessionSlot = TrackHttpHandle(session);

    DWORD secureProtocols =
        WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_2 |
        WINHTTP_FLAG_SECURE_PROTOCOL_TLS1_3;
    WinHttpSetOption(session, WINHTTP_OPTION_SECURE_PROTOCOLS, &secureProtocols, sizeof(secureProtocols));

    int resolveMs = 4000;
    int connectMs = 4000;
    int sendMs = 7000;
    int receiveMs = 20000;
    GetHttpTimeouts(displayName, url, resolveMs, connectMs, sendMs, receiveMs);
    WinHttpSetTimeouts(session, resolveMs, connectMs, sendMs, receiveMs);

    DWORD redirects = WINHTTP_OPTION_REDIRECT_POLICY_ALWAYS;
    WinHttpSetOption(session, WINHTTP_OPTION_REDIRECT_POLICY, &redirects, sizeof(redirects));

    connect = WinHttpConnect(session, p.host.c_str(), p.port, 0);

    if (!connect)
        return finish(MakeDownloadError(DownloadError::ConnectFailed, GetLastError(), true));

    connectSlot = TrackHttpHandle(connect);

    request = WinHttpOpenRequest(
        connect,
        L"GET",
        p.pathAndQuery.c_str(),
        nullptr,
        WINHTTP_NO_REFERER,
        WINHTTP_DEFAULT_ACCEPT_TYPES,
        p.https ? WINHTTP_FLAG_SECURE : 0
    );

    if (!request)
        return finish(MakeDownloadError(DownloadError::OpenRequestFailed, GetLastError(), true));

    requestSlot = TrackHttpHandle(request);

    std::wstring requestHeaders =
        L"User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36\r\n"
        L"Accept: application/octet-stream,application/zip,application/x-zip-compressed,application/x-msdownload,application/java-archive,*/*;q=0.8\r\n"
        L"Accept-Language: ru-RU,ru;q=0.9,en-US;q=0.7,en;q=0.6\r\n"
        L"Accept-Encoding: identity\r\n"
        L"Cache-Control: no-cache\r\n"
        L"Pragma: no-cache\r\n";

    if (ToLower(url).find(L"mods.holyworld.me") != std::wstring::npos)
        requestHeaders += L"Referer: https://mods.holyworld.me/applications\r\n";

    std::wstring partPath = targetPath + L".part";
    unsigned long long resumeFrom = GetFileSizeBytesOrZero(partPath);

    // Resume an interrupted .part file instead of starting from zero.
    if (resumeFrom > 0)
        requestHeaders += L"Range: bytes=" + std::to_wstring(resumeFrom) + L"-\r\n";

    WinHttpAddRequestHeaders(
        request,
        requestHeaders.c_str(),
        static_cast<DWORD>(-1),
        WINHTTP_ADDREQ_FLAG_ADD | WINHTTP_ADDREQ_FLAG_REPLACE
    );

    if (IsCancelRequested())
        return finish(MakeDownloadError(DownloadError::Cancelled, 0, false));

    if (!WinHttpSendRequest(request, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0))
        return finish(MakeDownloadError(DownloadError::SendFailed, GetLastError(), true));

    if (IsCancelRequested())
        return finish(MakeDownloadError(DownloadError::Cancelled, 0, false));

    if (!WinHttpReceiveResponse(request, nullptr))
        return finish(MakeDownloadError(DownloadError::ReceiveFailed, GetLastError(), true));

    if (IsCancelRequested())
        return finish(MakeDownloadError(DownloadError::Cancelled, 0, false));

    DWORD status = QueryStatus(request);

    if (resumeFrom > 0 && status == 416)
    {
        // Local .part is probably larger/equal to the remote object or the server
        // rejected the range. Reset it and let the retry start cleanly.
        DeleteFileW(partPath.c_str());
        DownloadResult result = MakeDownloadError(DownloadError::HttpStatus, 0, true);
        result.httpStatus = status;
        result.resumeFrom = resumeFrom;
        return finish(result);
    }

    if (status < 200 || status >= 300)
    {
        DownloadResult result = MakeDownloadError(DownloadError::HttpStatus, 0, IsRetryableHttpStatus(status));
        result.httpStatus = status;
        result.resumeFrom = resumeFrom;
        return finish(result);
    }

    bool appendToPart = resumeFrom > 0 && status == 206;

    // Some servers ignore Range and return 200. In that case restart safely.
    if (resumeFrom > 0 && status == 200)
    {
        DeleteFileW(partPath.c_str());
        resumeFrom = 0;
        appendToPart = false;
    }

    std::wstring contentType = QueryHeader(request, WINHTTP_QUERY_CONTENT_TYPE);

    if (LooksLikeHtml(contentType))
    {
        std::vector<unsigned char> html;

        if (!ReadAll(request, html, HTML_READ_LIMIT))
        {
            DownloadResult result = MakeDownloadError(DownloadError::ReadFailed, GetLastError(), true);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            return finish(result);
        }

        std::wstring direct = FindBinaryLink(url, html);

        if (direct.empty())
        {
            DownloadResult result = MakeDownloadError(DownloadError::HtmlLinkNotFound, 0, false);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            return finish(result);
        }

        CloseTrackedHttpHandle(requestSlot);
        CloseTrackedHttpHandle(connectSlot);
        CloseTrackedHttpHandle(sessionSlot);

        DownloadResult redirected = DownloadRecursiveOnce(direct, targetPath, displayName, depth + 1);
        if (redirected.finalUrl.empty())
            redirected.finalUrl = direct;
        return redirected;
    }

    unsigned long long expectedBytesThisResponse = QueryContentLength(request);

    HANDLE file = CreateFileW(
        partPath.c_str(),
        GENERIC_WRITE,
        0,
        nullptr,
        appendToPart ? OPEN_ALWAYS : CREATE_ALWAYS,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
    {
        DownloadResult result = MakeDownloadError(DownloadError::FileOpenFailed, GetLastError(), true);
        result.httpStatus = status;
        result.resumeFrom = resumeFrom;
        return finish(result);
    }

    if (appendToPart)
    {
        LARGE_INTEGER pos{};
        pos.QuadPart = static_cast<LONGLONG>(resumeFrom);
        if (!SetFilePointerEx(file, pos, nullptr, FILE_BEGIN))
        {
            DWORD err = GetLastError();
            CloseHandle(file);
            DownloadResult result = MakeDownloadError(DownloadError::FileOpenFailed, err, true);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            return finish(result);
        }
    }

    const DWORD BUFFER_SIZE = 1024 * 1024;
    std::vector<unsigned char> buffer(BUFFER_SIZE);
    unsigned long long bytesWrittenThisResponse = 0;

    for (;;)
    {
        if (IsCancelRequested())
        {
            CloseHandle(file);
            DownloadResult result = MakeDownloadError(DownloadError::Cancelled, 0, false);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            result.bytesWritten = bytesWrittenThisResponse;
            return finish(result);
        }

        DWORD available = 0;

        if (!WinHttpQueryDataAvailable(request, &available))
        {
            DWORD err = GetLastError();
            CloseHandle(file);
            DownloadResult result = MakeDownloadError(DownloadError::ReadFailed, err, true);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            result.bytesWritten = bytesWrittenThisResponse;
            return finish(result);
        }

        if (available == 0)
            break;

        DWORD toRead = std::min<DWORD>(available, BUFFER_SIZE);
        DWORD read = 0;

        if (!WinHttpReadData(request, buffer.data(), toRead, &read))
        {
            DWORD err = GetLastError();
            CloseHandle(file);
            DownloadResult result = MakeDownloadError(DownloadError::ReadFailed, err, true);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            result.bytesWritten = bytesWrittenThisResponse;
            return finish(result);
        }

        if (read == 0)
            break;

        DWORD written = 0;

        if (!WriteFile(file, buffer.data(), read, &written, nullptr) || written != read)
        {
            DWORD err = GetLastError();
            CloseHandle(file);
            DownloadResult result = MakeDownloadError(DownloadError::FileWriteFailed, err, true);
            result.httpStatus = status;
            result.resumeFrom = resumeFrom;
            result.bytesWritten = bytesWrittenThisResponse;
            return finish(result);
        }

        bytesWrittenThisResponse += written;
    }

    CloseHandle(file);

    if (expectedBytesThisResponse > 0 && bytesWrittenThisResponse < expectedBytesThisResponse)
    {
        DownloadResult result = MakeDownloadError(DownloadError::SizeMismatch, 0, true);
        result.httpStatus = status;
        result.contentLength = expectedBytesThisResponse;
        result.bytesWritten = bytesWrittenThisResponse;
        result.resumeFrom = resumeFrom;
        return finish(result);
    }

    if (IsCancelRequested())
    {
        DownloadResult result = MakeDownloadError(DownloadError::Cancelled, 0, false);
        result.httpStatus = status;
        result.resumeFrom = resumeFrom;
        result.bytesWritten = bytesWrittenThisResponse;
        return finish(result);
    }

    DeleteFileW(targetPath.c_str());

    if (!MoveFileW(partPath.c_str(), targetPath.c_str()))
    {
        DownloadResult result = MakeDownloadError(DownloadError::MoveFailed, GetLastError(), true);
        result.httpStatus = status;
        result.resumeFrom = resumeFrom;
        result.bytesWritten = bytesWrittenThisResponse;
        return finish(result);
    }

    DownloadResult ok = MakeDownloadOk();
    ok.httpStatus = status;
    ok.contentLength = expectedBytesThisResponse;
    ok.bytesWritten = bytesWrittenThisResponse;
    ok.resumeFrom = resumeFrom;
    ok.resumed = appendToPart;
    ok.finalUrl = url;
    return finish(ok);
}

static bool DownloadRecursive(
    const std::wstring& url,
    const std::wstring& targetPath,
    const std::wstring& displayName,
    int depth)
{
    const int maxAttempts = depth == 0 ? 4 : 3;
    DownloadResult lastResult;

    for (int attempt = 1; attempt <= maxAttempts; ++attempt)
    {
        if (attempt > 1 && !SleepRetryDelay(GetRetryDelayMs(attempt - 1)))
            return false;

        lastResult = DownloadRecursiveOnce(url, targetPath, displayName, depth);

        if (lastResult.ok)
        {
            if (lastResult.resumed)
                PostLog(L"Докачано: " + displayName);
            return true;
        }

        LogDownloadAttemptFailure(displayName, attempt, maxAttempts, lastResult);

        if (!lastResult.retryable || IsCancelRequested())
            break;
    }

    return false;
}

static bool FindFirstExeRecursive(const std::wstring& folder, std::wstring& outPath);

static bool ExtractZip(const std::wstring& zipPath, const std::wstring& destDir)
{
    std::lock_guard<std::mutex> lock(g_zipMutex);

    if (!FileExists(zipPath))
        return false;

    if (!EnsureDirectory(destDir))
        return false;

    HRESULT hr = CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED);
    bool needUninit = SUCCEEDED(hr);

    if (FAILED(hr) && hr != RPC_E_CHANGED_MODE)
        return false;

    IShellDispatch* shell = nullptr;
    hr = CoCreateInstance(CLSID_Shell, nullptr, CLSCTX_INPROC_SERVER, IID_PPV_ARGS(&shell));

    if (FAILED(hr) || !shell)
    {
        if (needUninit)
            CoUninitialize();

        return false;
    }

    VARIANT vZip;
    VARIANT vDest;

    VariantInit(&vZip);
    VariantInit(&vDest);

    vZip.vt = VT_BSTR;
    vZip.bstrVal = SysAllocString(zipPath.c_str());

    vDest.vt = VT_BSTR;
    vDest.bstrVal = SysAllocString(destDir.c_str());

    if (!vZip.bstrVal || !vDest.bstrVal)
    {
        VariantClear(&vZip);
        VariantClear(&vDest);
        shell->Release();

        if (needUninit)
            CoUninitialize();

        return false;
    }

    Folder* zipFolder = nullptr;
    Folder* destFolder = nullptr;

    shell->NameSpace(vZip, &zipFolder);
    shell->NameSpace(vDest, &destFolder);

    if (!zipFolder || !destFolder)
    {
        if (zipFolder) zipFolder->Release();
        if (destFolder) destFolder->Release();

        VariantClear(&vZip);
        VariantClear(&vDest);
        shell->Release();

        if (needUninit)
            CoUninitialize();

        return false;
    }

    FolderItems* items = nullptr;
    zipFolder->Items(&items);

    if (!items)
    {
        destFolder->Release();
        zipFolder->Release();
        VariantClear(&vZip);
        VariantClear(&vDest);
        shell->Release();

        if (needUninit)
            CoUninitialize();

        return false;
    }

    long itemCount = 0;
    items->get_Count(&itemCount);

    VARIANT vItems;
    VARIANT vOptions;

    VariantInit(&vItems);
    VariantInit(&vOptions);

    vItems.vt = VT_DISPATCH;
    vItems.pdispVal = items;

    // 4 = no progress UI, 16 = yes to all, 512 = do not confirm mkdir,
    // 1024 = do not show error UI.
    vOptions.vt = VT_I4;
    vOptions.lVal = 4 | 16 | 512 | 1024;

    hr = destFolder->CopyHere(vItems, vOptions);

    ULONGLONG started = GetTickCount64();
    bool copiedSomething = itemCount <= 0;

    while (SUCCEEDED(hr) && !IsCancelRequested() && GetTickCount64() - started < ZIP_EXTRACT_TIMEOUT_MS)
    {
        std::wstring exePath;

        if (FindFirstExeRecursive(destDir, exePath))
        {
            copiedSomething = true;
            break;
        }

        WIN32_FIND_DATAW data{};
        std::wstring mask = CombinePath(destDir, L"*");
        HANDLE hFind = FindFirstFileW(mask.c_str(), &data);

        if (hFind != INVALID_HANDLE_VALUE)
        {
            do
            {
                if (wcscmp(data.cFileName, L".") != 0 && wcscmp(data.cFileName, L"..") != 0)
                {
                    copiedSomething = true;
                    break;
                }
            }
            while (FindNextFileW(hFind, &data));

            FindClose(hFind);
        }

        if (copiedSomething && GetTickCount64() - started >= 1500)
            break;

        Sleep(ZIP_EXTRACT_POLL_MS);
    }

    items->Release();
    destFolder->Release();
    zipFolder->Release();

    VariantClear(&vZip);
    VariantClear(&vDest);
    shell->Release();

    if (needUninit)
        CoUninitialize();

    return SUCCEEDED(hr) && copiedSomething && !IsCancelRequested();
}

static std::wstring EscapePowerShellSingleQuoted(std::wstring s)
{
    size_t pos = 0;

    while ((pos = s.find(L'\'', pos)) != std::wstring::npos)
    {
        s.insert(pos, L"'");
        pos += 2;
    }

    return s;
}
static std::wstring QuoteProcessArgument(const std::wstring& arg)
{
    if (arg.empty())
        return L"\"\"";

    bool needQuotes = false;

    for (wchar_t ch : arg)
    {
        if (iswspace(ch) || ch == L'\"')
        {
            needQuotes = true;
            break;
        }
    }

    if (!needQuotes)
        return arg;

    std::wstring out = L"\"";
    size_t backslashes = 0;

    for (wchar_t ch : arg)
    {
        if (ch == L'\\')
        {
            ++backslashes;
        }
        else if (ch == L'\"')
        {
            out.append(backslashes * 2 + 1, L'\\');
            out.push_back(ch);
            backslashes = 0;
        }
        else
        {
            out.append(backslashes, L'\\');
            backslashes = 0;
            out.push_back(ch);
        }
    }

    out.append(backslashes * 2, L'\\');
    out.push_back(L'\"');
    return out;
}

static std::wstring GetSystemToolPath(const wchar_t* relativePath)
{
    wchar_t systemDir[MAX_PATH * 2]{};
    UINT len = GetSystemDirectoryW(systemDir, ARRAYSIZE(systemDir));

    if (len == 0 || len >= ARRAYSIZE(systemDir))
        return relativePath ? relativePath : L"";

    std::wstring path = CombinePath(systemDir, relativePath ? relativePath : L"");
    return FileExists(path) ? path : std::wstring(relativePath ? relativePath : L"");
}

static std::wstring Base64EncodeBytes(const unsigned char* data, size_t len)
{
    static const wchar_t alphabet[] = L"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    std::wstring out;
    out.reserve(((len + 2) / 3) * 4);

    for (size_t i = 0; i < len; i += 3)
    {
        unsigned int value = data[i] << 16;
        bool has2 = i + 1 < len;
        bool has3 = i + 2 < len;

        if (has2)
            value |= data[i + 1] << 8;

        if (has3)
            value |= data[i + 2];

        out.push_back(alphabet[(value >> 18) & 0x3F]);
        out.push_back(alphabet[(value >> 12) & 0x3F]);
        out.push_back(has2 ? alphabet[(value >> 6) & 0x3F] : L'=');
        out.push_back(has3 ? alphabet[value & 0x3F] : L'=');
    }

    return out;
}

static std::wstring Base64EncodePowerShellCommand(const std::wstring& command)
{
    return Base64EncodeBytes(reinterpret_cast<const unsigned char*>(command.data()), command.size() * sizeof(wchar_t));
}



static bool DownloadWithCurlFallback(const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs = 18000);
static bool DownloadWithPowerShellFallback(const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs = 22000);
static bool DownloadRecursiveValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath);
static bool DownloadCurlValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath);
static bool DownloadPowerShellValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath);

static std::wstring NormalizeSoftPortalSearchTerm(std::wstring s)
{
    for (wchar_t& ch : s)
    {
        if (ch == L'-' || ch == L'_' || ch == L'.' || ch == L'+' || ch == L'(' || ch == L')')
            ch = L' ';
    }

    while (!s.empty() && iswspace(s.front()))
        s.erase(s.begin());

    while (!s.empty() && iswspace(s.back()))
        s.pop_back();

    std::wstring out;
    bool lastSpace = false;

    for (wchar_t ch : s)
    {
        bool space = iswspace(ch) != 0;

        if (space)
        {
            if (!lastSpace)
                out.push_back(L' ');
        }
        else
        {
            out.push_back(ch);
        }

        lastSpace = space;
    }

    return out;
}

static void AddUniqueString(std::vector<std::wstring>& values, const std::wstring& value)
{
    if (value.empty())
        return;

    std::wstring lowerValue = ToLower(value);

    for (const auto& existing : values)
    {
        if (ToLower(existing) == lowerValue)
            return;
    }

    values.push_back(value);
}

static std::vector<std::wstring> GetSoftPortalSearchTerms(const DownloadItem& item)
{
    std::vector<std::wstring> terms;

    AddUniqueString(terms, item.name);
    AddUniqueString(terms, NormalizeSoftPortalSearchTerm(item.name));
    AddUniqueString(terms, RemoveExtension(item.file));
    AddUniqueString(terms, NormalizeSoftPortalSearchTerm(RemoveExtension(item.file)));

    std::wstring lowerName = ToLower(item.name);

    if (lowerName.find(L"system") != std::wstring::npos && lowerName.find(L"informer") != std::wstring::npos)
    {
        AddUniqueString(terms, L"System Informer");
        AddUniqueString(terms, L"Process Hacker");
    }
    else if (lowerName.find(L"browserdownloads") != std::wstring::npos)
    {
        AddUniqueString(terms, L"Browser Downloads View");
    }
    else if (lowerName.find(L"browsinghistory") != std::wstring::npos)
    {
        AddUniqueString(terms, L"Browsing History View");
    }
    else if (lowerName.find(L"recentfiles") != std::wstring::npos)
    {
        AddUniqueString(terms, L"Recent Files View");
    }
    else if (lowerName.find(L"executedprograms") != std::wstring::npos)
    {
        AddUniqueString(terms, L"Executed Programs List");
    }
    else if (lowerName.find(L"usbdeview") != std::wstring::npos)
    {
        AddUniqueString(terms, L"USBDeview");
        AddUniqueString(terms, L"USB Deview");
    }
    else if (lowerName.find(L"usbdrivelog") != std::wstring::npos)
    {
        AddUniqueString(terms, L"USB Drive Log");
    }
    else if (lowerName.find(L"windeflog") != std::wstring::npos)
    {
        AddUniqueString(terms, L"WinDefLogView");
        AddUniqueString(terms, L"WinDef Log View");
    }
    else if (lowerName.find(L"alternatestream") != std::wstring::npos)
    {
        AddUniqueString(terms, L"AlternateStreamView");
        AddUniqueString(terms, L"Alternate Stream View");
    }
    else if (lowerName.find(L"winprefetch") != std::wstring::npos)
    {
        AddUniqueString(terms, L"WinPrefetchView");
        AddUniqueString(terms, L"Win Prefetch View");
    }
    else if (lowerName.find(L"winrar") != std::wstring::npos)
    {
        AddUniqueString(terms, L"WinRAR");
    }
    else if (lowerName.find(L"recuva") != std::wstring::npos)
    {
        AddUniqueString(terms, L"Recuva");
        AddUniqueString(terms, L"Piriform Recuva");
    }

    return terms;
}

static std::vector<std::wstring> GetSoftPortalFallbackUrls(const DownloadItem& item)
{
    std::vector<std::wstring> urls;
    std::wstring name = ToLower(item.name);

    // Stable known SoftPortal pages for programs that do not always have direct CDN links.
    if (name.find(L"recuva") != std::wstring::npos)
    {
        urls.push_back(L"https://www.softportal.com/getsoft-5667-recuva-3.html");
        urls.push_back(L"https://www.softportal.com/get-5667-recuva.html");
        urls.push_back(L"https://www.softportal.com/en/recuva/windows/download");
    }

    std::vector<std::wstring> terms = GetSoftPortalSearchTerms(item);

    // Generic SoftPortal fallback is intentionally compact: if the official URL is down,
    // try the strongest 2 names and 2 endpoints instead of crawling many pages slowly.
    int termCount = 0;
    for (const auto& term : terms)
    {
        if (termCount >= 2)
            break;

        std::wstring q = UrlEncodeUtf8(term);

        AddUniqueString(urls, L"https://www.softportal.com/search.html?text=" + q);
        AddUniqueString(urls, L"https://www.softportal.com/en/search?q=" + q);
        ++termCount;
    }

    return urls;
}



static std::string AsciiLower(std::string value)
{
    std::transform(value.begin(), value.end(), value.begin(), [](unsigned char c)
    {
        return static_cast<char>(std::tolower(c));
    });
    return value;
}

static std::string WideLowerUtf8(const std::wstring& value)
{
    return AsciiLower(WideToUtf8(ToLower(value)));
}

static bool StringContainsAny(const std::string& text, const std::vector<std::string>& needles)
{
    for (const auto& needle : needles)
    {
        if (!needle.empty() && text.find(needle) != std::string::npos)
            return true;
    }

    return false;
}

static std::vector<std::string> GetItemSearchNeedles(const DownloadItem& item)
{
    std::vector<std::string> needles;
    auto add = [&](std::string v)
    {
        v = AsciiLower(v);
        if (!v.empty() && std::find(needles.begin(), needles.end(), v) == needles.end())
            needles.push_back(v);
    };

    add(WideToUtf8(item.name));
    add(WideToUtf8(RemoveExtension(item.file)));

    std::string compactName = WideToUtf8(item.name);
    compactName.erase(std::remove_if(compactName.begin(), compactName.end(), [](unsigned char c)
    {
        return c == ' ' || c == '-' || c == '_' || c == '.';
    }), compactName.end());
    add(compactName);

    std::wstring lower = ToLower(item.name);

    if (IsRegistryExplorerItem(item))
    {
        add("registryexplorer");
        add("registry explorer");
        add("registry");
    }
    else if (IsJumpListExplorerItem(item))
    {
        add("jumplistexplorer");
        add("jumplist explorer");
        add("jumplist");
        add("jump list");
        add("jumplistexplore");
    }
    else if (IsSimpleUnlockerItem(item))
    {
        add("simpleunlocker");
        add("simple unlocker");
        add("simpleunlocker_release");
        add("unlocker");
    }
    else if (IsWinRarItem(item))
    {
        add("winrar");
        add("win-rar");
        add("rarlab");
    }
    else if (IsSystemInformerItem(item))
    {
        add("systeminformer");
        add("system informer");
    }

    return needles;
}

static bool IsDownloadLikeUrl(const std::string& lowerUrl)
{
    return lowerUrl.find(".zip") != std::string::npos ||
        lowerUrl.find(".exe") != std::string::npos ||
        lowerUrl.find(".jar") != std::string::npos ||
        lowerUrl.find(".7z") != std::string::npos ||
        lowerUrl.find(".rar") != std::string::npos ||
        lowerUrl.find(".msi") != std::string::npos ||
        lowerUrl.find("/download") != std::string::npos ||
        lowerUrl.find("download?") != std::string::npos ||
        lowerUrl.find("/api/") != std::string::npos;
}

static std::wstring FindBestDownloadLinkForItem(
    const std::wstring& baseUrl,
    const std::vector<unsigned char>& bytes,
    const DownloadItem& item)
{
    std::string html(bytes.begin(), bytes.end());
    std::string lowerHtml = AsciiLower(html);
    std::vector<std::string> needles = GetItemSearchNeedles(item);

    int bestScore = 0;
    std::wstring bestUrl;

    auto scoreCandidate = [&](const std::string& rawValue, size_t pos)
    {
        if (rawValue.empty())
            return;

        std::string raw = rawValue;
        while (!raw.empty() && (raw.front() == '\'' || raw.front() == '"' || raw.front() == '`' || raw.front() == '('))
            raw.erase(raw.begin());
        while (!raw.empty() && (raw.back() == '\'' || raw.back() == '"' || raw.back() == '`' || raw.back() == ',' || raw.back() == ';' || raw.back() == ')' || raw.back() == ']'))
            raw.pop_back();

        std::string lowerRaw = AsciiLower(raw);

        if (lowerRaw.empty() || lowerRaw == "#" || lowerRaw.rfind("javascript:", 0) == 0 || lowerRaw.rfind("mailto:", 0) == 0)
            return;

        bool maybeDownloadRoute =
            lowerRaw.find("download") != std::string::npos ||
            lowerRaw.find("application") != std::string::npos ||
            lowerRaw.find("file") != std::string::npos ||
            lowerRaw.find("api") != std::string::npos ||
            IsDownloadLikeUrl(lowerRaw);

        if (!maybeDownloadRoute)
            return;

        std::wstring absolute = MakeAbsoluteUrl(baseUrl, CleanHtmlUrl(raw));
        std::string absoluteLower = WideLowerUtf8(absolute);

        size_t ctxStart = pos > 1800 ? pos - 1800 : 0;
        size_t ctxEnd = std::min(html.size(), pos + raw.size() + 1800);
        std::string context = lowerHtml.substr(ctxStart, ctxEnd - ctxStart);

        if (absoluteLower == "https://mods.holyworld.me/applications" ||
            absoluteLower == "https://mods.holyworld.me/download" ||
            absoluteLower.find("/login") != std::string::npos ||
            absoluteLower.find("/staff") != std::string::npos ||
            absoluteLower.find("/news") != std::string::npos ||
            absoluteLower.find("/mods-review") != std::string::npos)
        {
            return;
        }

        bool externalHttp = absoluteLower.find("mods.holyworld.me") == std::string::npos && absoluteLower.rfind("http", 0) == 0;
        if (externalHttp && !StringContainsAny(absoluteLower, needles) && !StringContainsAny(context, needles))
            return;

        int score = 0;

        if (StringContainsAny(absoluteLower, needles))
            score += 130;

        if (StringContainsAny(context, needles))
            score += 110;

        if (absoluteLower.find("mods.holyworld.me") != std::string::npos)
            score += 45;

        if (IsDownloadLikeUrl(absoluteLower))
            score += 45;

        if (absoluteLower.find("download") != std::string::npos)
            score += 35;

        if (absoluteLower.find("/api/") != std::string::npos)
            score += 20;

        if (absoluteLower.find(".zip") != std::string::npos || absoluteLower.find(".exe") != std::string::npos)
            score += 35;

        if (absoluteLower.find("login") != std::string::npos || absoluteLower.find("logout") != std::string::npos || absoluteLower.find("register") != std::string::npos)
            score -= 200;

        if (score > bestScore && score >= 125)
        {
            bestScore = score;
            bestUrl = absolute;
        }
    };

    std::regex attrRe(
        R"HC((?:href|src|data-url|data-href|data-link|data-download|data-download-url|download-url)\s*=\s*["']([^"']+)["'])HC",
        std::regex_constants::icase
    );

    std::smatch m;
    auto begin = html.cbegin();
    auto end = html.cend();

    while (std::regex_search(begin, end, m, attrRe))
    {
        size_t pos = static_cast<size_t>(m.position(1) + std::distance(html.cbegin(), begin));
        scoreCandidate(m[1].str(), pos);
        begin = m.suffix().first;
    }

    std::regex jsonUrlRe(
        R"HC((?:"|')?(?:url|href|link|file|path|download|downloadUrl|download_url|downloadLink|download_link)(?:"|')?\s*[:=]\s*(?:"|')([^"']+)(?:"|'))HC",
        std::regex_constants::icase
    );

    begin = html.cbegin();
    while (std::regex_search(begin, end, m, jsonUrlRe))
    {
        size_t pos = static_cast<size_t>(m.position(1) + std::distance(html.cbegin(), begin));
        scoreCandidate(m[1].str(), pos);
        begin = m.suffix().first;
    }

    std::regex plainUrlRe(
        R"HC((https?://[^\s"'<>`]+(?:\.zip|\.exe|\.jar|\.7z|\.rar|\.msi|download|api|applications)[^\s"'<>`]*))HC",
        std::regex_constants::icase
    );

    begin = html.cbegin();
    while (std::regex_search(begin, end, m, plainUrlRe))
    {
        size_t pos = static_cast<size_t>(m.position(1) + std::distance(html.cbegin(), begin));
        scoreCandidate(m[1].str(), pos);
        begin = m.suffix().first;
    }

    std::regex relativeRe(
        R"HC((/(?:api/)?(?:applications?|download|downloads|files)[^\s"'<>`]+))HC",
        std::regex_constants::icase
    );

    begin = html.cbegin();
    while (std::regex_search(begin, end, m, relativeRe))
    {
        size_t pos = static_cast<size_t>(m.position(1) + std::distance(html.cbegin(), begin));
        scoreCandidate(m[1].str(), pos);
        begin = m.suffix().first;
    }

    return bestUrl;
}


static bool FetchUrlBytes(const std::wstring& url, const std::wstring& displayName, std::vector<unsigned char>& out)
{
    out.clear();

    if (IsCancelRequested())
        return false;

    ParsedUrl p;
    if (!ParseUrl(url, p))
        return false;

    HINTERNET session = nullptr;
    HINTERNET connect = nullptr;
    HINTERNET request = nullptr;
    std::shared_ptr<HttpHandleSlot> sessionSlot;
    std::shared_ptr<HttpHandleSlot> connectSlot;
    std::shared_ptr<HttpHandleSlot> requestSlot;
    bool ok = false;

    session = WinHttpOpen(
        L"Mozilla/5.0 HolyCheckDownloader/9.2",
        WINHTTP_ACCESS_TYPE_DEFAULT_PROXY,
        WINHTTP_NO_PROXY_NAME,
        WINHTTP_NO_PROXY_BYPASS,
        0
    );

    if (!session)
        return false;

    sessionSlot = TrackHttpHandle(session);

    int resolveMs = 3500;
    int connectMs = 3500;
    int sendMs = 5000;
    int receiveMs = 12000;
    GetHttpTimeouts(displayName, url, resolveMs, connectMs, sendMs, receiveMs);
    WinHttpSetTimeouts(session, resolveMs, connectMs, sendMs, std::min(receiveMs, 12000));

    DWORD redirects = WINHTTP_OPTION_REDIRECT_POLICY_ALWAYS;
    WinHttpSetOption(session, WINHTTP_OPTION_REDIRECT_POLICY, &redirects, sizeof(redirects));

    connect = WinHttpConnect(session, p.host.c_str(), p.port, 0);
    if (!connect)
        goto cleanup;

    connectSlot = TrackHttpHandle(connect);

    request = WinHttpOpenRequest(
        connect,
        L"GET",
        p.pathAndQuery.c_str(),
        nullptr,
        WINHTTP_NO_REFERER,
        WINHTTP_DEFAULT_ACCEPT_TYPES,
        p.https ? WINHTTP_FLAG_SECURE : 0
    );

    if (!request)
        goto cleanup;

    requestSlot = TrackHttpHandle(request);

    WinHttpAddRequestHeaders(
        request,
        L"User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36\r\n"
        L"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,application/json,*/*;q=0.8\r\n"
        L"Accept-Language: ru-RU,ru;q=0.9,en-US;q=0.7,en;q=0.6\r\n"
        L"Referer: https://mods.holyworld.me/applications\r\n"
        L"Cache-Control: no-cache\r\n"
        L"Pragma: no-cache\r\n",
        static_cast<DWORD>(-1),
        WINHTTP_ADDREQ_FLAG_ADD | WINHTTP_ADDREQ_FLAG_REPLACE
    );

    if (!WinHttpSendRequest(request, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0))
        goto cleanup;

    if (!WinHttpReceiveResponse(request, nullptr))
        goto cleanup;

    {
        DWORD status = QueryStatus(request);
        if (status < 200 || status >= 300)
            goto cleanup;
    }

    ok = ReadAll(request, out, HTML_READ_LIMIT) && !out.empty();

cleanup:
    CloseTrackedHttpHandle(requestSlot);
    CloseTrackedHttpHandle(connectSlot);
    CloseTrackedHttpHandle(sessionSlot);
    return ok && !IsCancelRequested();
}

static std::vector<std::wstring> GetHolyWorldCandidateUrls(const DownloadItem& item)
{
    std::vector<std::wstring> urls;

    auto addFor = [&](const std::wstring& canonical, const std::wstring& file)
    {
        std::wstring encodedCanonical = UrlEncodeUtf8(canonical);
        std::wstring encodedFile = UrlEncodeUtf8(file);

        // Keep this list short: first parse the real site page, then try only likely direct routes.
        AddUniqueString(urls, L"https://mods.holyworld.me/download/" + canonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/download/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications/download/" + canonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications/download/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/download/" + canonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/download/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/applications/download/" + canonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/applications/download/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/download?name=" + encodedCanonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/download?file=" + encodedFile);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications?download=" + encodedCanonical);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications?file=" + encodedFile);
        AddUniqueString(urls, L"https://mods.holyworld.me/files/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/downloads/" + file);
        AddUniqueString(urls, L"https://mods.holyworld.me/storage/applications/" + file);
    };

    if (IsRegistryExplorerItem(item))
    {
        addFor(L"RegistryExplorer", L"RegistryExplorer.zip");
        addFor(L"registryexplorer", L"registryexplorer.zip");
        addFor(L"registry-explorer", L"RegistryExplorer.zip");
    }
    else if (IsJumpListExplorerItem(item))
    {
        addFor(L"JumpListExplorer", L"JumpListExplorer.zip");
        addFor(L"JumpListExplore", L"JumpListExplorer.zip");
        addFor(L"jumplistexplorer", L"jumplistexplorer.zip");
        addFor(L"jumplist-explorer", L"JumpListExplorer.zip");
    }
    else if (IsSimpleUnlockerItem(item))
    {
        addFor(L"SimpleUnlocker", L"simpleunlocker_release.zip");
        addFor(L"Simpleunlocker", L"simpleunlocker_release.zip");
        addFor(L"simpleunlocker", L"simpleunlocker_release.zip");
        addFor(L"simple-unlocker", L"simpleunlocker_release.zip");
    }

    return urls;
}

static std::vector<std::wstring> FindHolyWorldCandidateUrlsInPage(
    const std::wstring& baseUrl,
    const std::vector<unsigned char>& bytes,
    const DownloadItem& item)
{
    std::vector<std::wstring> urls;
    std::string html(bytes.begin(), bytes.end());
    std::string lowerHtml = AsciiLower(html);
    std::vector<std::string> needles = GetItemSearchNeedles(item);

    auto addRoutesForValue = [&](const std::string& value)
    {
        if (value.empty() || value.size() > 96)
            return;

        std::string v = value;
        while (!v.empty() && (v.front() == '\'' || v.front() == '"' || v.front() == '`' || v.front() == '('))
            v.erase(v.begin());
        while (!v.empty() && (v.back() == '\'' || v.back() == '"' || v.back() == '`' || v.back() == ',' || v.back() == ';' || v.back() == ')' || v.back() == ']'))
            v.pop_back();

        if (v.empty() || v == "true" || v == "false" || v == "null" || v == "download" || v == "applications")
            return;

        std::wstring w = Utf8ToWide(v);
        std::wstring e = UrlEncodeUtf8(w);
        std::string lowerValue = AsciiLower(v);

        if (lowerValue.rfind("http://", 0) == 0 || lowerValue.rfind("https://", 0) == 0 || lowerValue.rfind("/", 0) == 0)
        {
            AddUniqueString(urls, MakeAbsoluteUrl(baseUrl, w));
            return;
        }

        AddUniqueString(urls, L"https://mods.holyworld.me/download/" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications/download/" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/download/" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/api/applications/download/" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/download?id=" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/download?name=" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/download?file=" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications?id=" + e);
        AddUniqueString(urls, L"https://mods.holyworld.me/applications?download=" + e);

        if (lowerValue.find(".zip") != std::string::npos || lowerValue.find(".exe") != std::string::npos)
        {
            AddUniqueString(urls, L"https://mods.holyworld.me/files/" + w);
            AddUniqueString(urls, L"https://mods.holyworld.me/downloads/" + w);
            AddUniqueString(urls, L"https://mods.holyworld.me/storage/applications/" + w);
        }
    };

    std::regex keyValueRe(
        R"HC((?:\"|')?(?:id|_id|appId|applicationId|slug|key|name|file|filename|download|downloadUrl|download_url)(?:\"|')?\s*[:=]\s*(?:\"|')?([A-Za-z0-9_.\-]{1,96})(?:\"|')?)HC",
        std::regex_constants::icase
    );

    for (const auto& needle : needles)
    {
        if (needle.empty())
            continue;

        size_t pos = lowerHtml.find(needle);
        while (pos != std::string::npos)
        {
            size_t ctxStart = pos > 2200 ? pos - 2200 : 0;
            size_t ctxEnd = std::min(html.size(), pos + needle.size() + 2200);
            std::string ctx = html.substr(ctxStart, ctxEnd - ctxStart);

            std::smatch m;
            auto begin = ctx.cbegin();
            auto end = ctx.cend();
            while (std::regex_search(begin, end, m, keyValueRe))
            {
                addRoutesForValue(m[1].str());
                begin = m.suffix().first;
            }

            pos = lowerHtml.find(needle, pos + needle.size());
        }
    }

    return urls;
}

static bool TryHolyWorldDownloadUrl(const DownloadItem& item, const std::wstring& targetPath, const std::wstring& url, bool allowPowerShell)
{
    if (IsCancelRequested() || url.empty())
        return false;

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());

    if (DownloadRecursiveValidated(item, url, targetPath))
        return true;

    if (DownloadCurlValidated(item, url, targetPath))
        return true;

    if (allowPowerShell && DownloadPowerShellValidated(item, url, targetPath))
        return true;

    return false;
}

static bool TryHolyWorldFallback(const DownloadItem& item, const std::wstring& targetPath)
{
    if (!(IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item) || IsSimpleUnlockerItem(item)))
        return false;

    PostLog(L"HolyWorld: " + item.name);

    const std::wstring pages[] =
    {
        L"https://mods.holyworld.me/applications",
        L"https://mods.holyworld.me/download"
    };

    for (const auto& page : pages)
    {
        if (IsCancelRequested())
            return false;

        std::vector<unsigned char> html;
        if (!FetchUrlBytes(page, item.name, html))
            continue;

        std::wstring direct = FindBestDownloadLinkForItem(page, html, item);
        if (!direct.empty())
        {
            PostLog(L"HolyWorld link: " + item.name);
            if (TryHolyWorldDownloadUrl(item, targetPath, direct, true))
                return true;
        }

        std::vector<std::wstring> pageUrls = FindHolyWorldCandidateUrlsInPage(page, html, item);
        for (const auto& url : pageUrls)
        {
            if (IsCancelRequested())
                return false;

            if (TryHolyWorldDownloadUrl(item, targetPath, url, false))
                return true;
        }
    }

    std::vector<std::wstring> candidates = GetHolyWorldCandidateUrls(item);
    for (const auto& url : candidates)
    {
        if (IsCancelRequested())
            return false;

        if (TryHolyWorldDownloadUrl(item, targetPath, url, false))
            return true;
    }

    return false;
}


static std::vector<std::wstring> GetFastFallbackUrls(const DownloadItem& item)
{
    std::vector<std::wstring> urls;
    std::wstring name = ToLower(item.name);

    if (IsSystemInformerItem(item))
    {
        // Сначала старая рабочая прямая ссылка, затем страницы/зеркала.
        AddUniqueString(urls, L"https://downloads.sourceforge.net/project/systeminformer/systeminformer-3.2.25011-release-setup.exe");
        AddUniqueString(urls, L"https://sourceforge.net/projects/systeminformer/files/systeminformer-3.2.25011-release-setup.exe/download");
        AddUniqueString(urls, L"https://github.com/winsiderss/systeminformer/releases/download/v3.2.25011.2103/systeminformer-3.2.25011-release-setup.exe");
        AddUniqueString(urls, L"https://systeminformer.sourceforge.io/canary/systeminformer-3.2.25011-setup.exe");
    }
    else if (IsWinRarItem(item))
    {
        // Прямая ссылка RARLAB на русский x64 installer, если postdownload вернул HTML.
        AddUniqueString(urls, L"https://www.win-rar.com/postdownload.html?&L=4");
        AddUniqueString(urls, L"https://www.rarlab.com/rar/winrar-x64-722ru.exe");
        AddUniqueString(urls, L"https://www.rarlab.com/rar/winrar-x64-722.exe");
        AddUniqueString(urls, L"https://www.rarlab.com/rar/winrar-x64-721ru.exe");
        AddUniqueString(urls, L"https://www.rarlab.com/rar/winrar-x64-721.exe");
    }
    else if (name.find(L"recuva") != std::wstring::npos)
    {
        AddUniqueString(urls, L"https://www.softportal.com/getsoft-5667-recuva-3.html");
        AddUniqueString(urls, L"https://download.ccleaner.com/rcsetup154.exe");
        AddUniqueString(urls, L"https://www.ccleaner.com/recuva/download/standard");
        AddUniqueString(urls, L"https://sourceforge.net/projects/recuva/files/rcsetup154.exe/download");
        AddUniqueString(urls, L"https://downloads.sourceforge.net/project/recuva/rcsetup154.exe");
    }

    return urls;
}

static bool TryFastFallbackUrls(const DownloadItem& item, const std::wstring& targetPath)
{
    std::vector<std::wstring> urls = GetFastFallbackUrls(item);

    for (const auto& url : urls)
    {
        if (IsCancelRequested())
            return false;

        if (ToLower(url) == ToLower(item.url))
            continue;

        DeleteFileW(targetPath.c_str());
        DeleteFileW((targetPath + L".part").c_str());

        PostLog(L"Быстрый резерв: " + item.name);

        if (IsEricZimmermanLargeItem(item))
        {
            // For large Eric Zimmerman archives curl handles CDN redirects and
            // slow streams closer to how the browser behaves, so try it first.
            if (DownloadCurlValidated(item, url, targetPath))
                return true;

            if (IsCancelRequested())
                return false;

            if (DownloadRecursiveValidated(item, url, targetPath))
                return true;
        }
        else
        {
            if (DownloadRecursiveValidated(item, url, targetPath))
                return true;

            if (IsCancelRequested())
                return false;

            if (DownloadCurlValidated(item, url, targetPath))
                return true;
        }

        if (IsCancelRequested())
            return false;

        if (DownloadPowerShellValidated(item, url, targetPath))
            return true;
    }

    return false;
}

static bool TrySoftPortalFallback(const DownloadItem& item, const std::wstring& targetPath)
{
    if (IsCancelRequested() || IsFastFailItem(item))
        return false;

    PostLog(L"Альтернатива: поиск " + item.name);

    std::vector<std::wstring> urls = GetSoftPortalFallbackUrls(item);

    for (const auto& url : urls)
    {
        if (IsCancelRequested())
            return false;

        DeleteFileW(targetPath.c_str());
        DeleteFileW((targetPath + L".part").c_str());

        if (DownloadRecursiveValidated(item, url, targetPath))
        {
            PostLog(L"Альтернатива: скачано " + item.name);
            return true;
        }

        if (IsCancelRequested())
            return false;

        if (DownloadPowerShellValidated(item, url, targetPath))
        {
            PostLog(L"Альтернатива: скачано " + item.name);
            return true;
        }
    }

    PostLog(L"Альтернатива: не найдена " + item.name);
    return false;
}

static bool DownloadWithCurlFallback(const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs)
{
    if (IsCancelRequested())
        return false;

    std::wstring partPath = targetPath + L".part";

    std::wstring curlExe = GetSystemToolPath(L"curl.exe");
    DWORD maxTimeSeconds = std::max<DWORD>(8, timeoutMs / 1000);
    DWORD connectTimeoutSeconds = timeoutMs >= 300000 ? 15 : 6;

    std::wstring command =
        QuoteProcessArgument(curlExe) +
        L" -L --fail --silent --show-error --http1.1 --ipv4 --ssl-no-revoke --tcp-nodelay"
        L" -C -"
        L" --retry 1 --retry-delay 1 --retry-connrefused"
        L" --speed-limit 2048 --speed-time 30"
        L" --connect-timeout " + std::to_wstring(connectTimeoutSeconds) +
        L" --max-time " + std::to_wstring(maxTimeSeconds) +
        L" -A " + QuoteProcessArgument(L"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36") +
        L" -H " + QuoteProcessArgument(L"Accept: application/octet-stream,*/*;q=0.8") +
        L" -H " + QuoteProcessArgument(L"Accept-Language: ru-RU,ru;q=0.9,en-US;q=0.7,en;q=0.6") +
        L" -o " + QuoteProcessArgument(partPath) +
        L" " + QuoteProcessArgument(url);

    STARTUPINFOW si{};
    si.cb = sizeof(si);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

    PROCESS_INFORMATION pi{};

    std::vector<wchar_t> cmd(command.begin(), command.end());
    cmd.push_back(L'\0');

    const wchar_t* appName = FileExists(curlExe) ? curlExe.c_str() : nullptr;

    if (!CreateProcessW(appName, cmd.data(), nullptr, nullptr, FALSE, CREATE_NO_WINDOW, nullptr, nullptr, &si, &pi))
        return false;

    ULONGLONG started = GetTickCount64();

    for (;;)
    {
        DWORD waitResult = WaitForSingleObject(pi.hProcess, 150);

        if (waitResult == WAIT_OBJECT_0)
            break;

        if (waitResult == WAIT_FAILED || IsCancelRequested() || GetTickCount64() - started > timeoutMs)
        {
            TerminateProcess(pi.hProcess, 1);
            WaitForSingleObject(pi.hProcess, 1000);
            CloseHandle(pi.hThread);
            CloseHandle(pi.hProcess);
            return false;
        }
    }

    DWORD exitCode = 1;
    GetExitCodeProcess(pi.hProcess, &exitCode);

    CloseHandle(pi.hThread);
    CloseHandle(pi.hProcess);

    if (exitCode != 0 || !FileExists(partPath) || IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());

    if (!MoveFileW(partPath.c_str(), targetPath.c_str()))
        return false;

    return true;
}


static bool DownloadWithCurlBrowserLike(const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs)
{
    if (IsCancelRequested())
        return false;

    std::wstring partPath = targetPath + L".part";

    std::wstring curlExe = GetSystemToolPath(L"curl.exe");
    DWORD maxTimeSeconds = std::max<DWORD>(300, timeoutMs / 1000);

    // Browser-like large archive download. Do not use an aggressive speed-limit
    // here: Eric Zimmerman CDN can start slowly on some networks, and the old
    // 64 KB/s rule aborted valid downloads and then printed "direct links failed".
    // Keep IPv4 and ssl-no-revoke because they reduce common Windows curl stalls.
    std::wstring command =
        QuoteProcessArgument(curlExe) +
        L" -L --fail --silent --show-error --http1.1 --ipv4 --ssl-no-revoke --tcp-nodelay"
        L" -C -"
        L" --retry 3 --retry-delay 2 --retry-connrefused"
        L" --connect-timeout 15"
        L" --max-time " + std::to_wstring(maxTimeSeconds) +
        L" -A " + QuoteProcessArgument(L"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0 Safari/537.36") +
        L" -H " + QuoteProcessArgument(L"Accept: application/octet-stream,*/*;q=0.8") +
        L" -H " + QuoteProcessArgument(L"Accept-Language: ru-RU,ru;q=0.9,en-US;q=0.7,en;q=0.6") +
        L" -e " + QuoteProcessArgument(L"https://ericzimmerman.github.io/") +
        L" -o " + QuoteProcessArgument(partPath) +
        L" " + QuoteProcessArgument(url);

    STARTUPINFOW si{};
    si.cb = sizeof(si);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

    PROCESS_INFORMATION pi{};

    std::vector<wchar_t> cmd(command.begin(), command.end());
    cmd.push_back(L'\0');

    const wchar_t* appName = FileExists(curlExe) ? curlExe.c_str() : nullptr;

    if (!CreateProcessW(appName, cmd.data(), nullptr, nullptr, FALSE, CREATE_NO_WINDOW, nullptr, nullptr, &si, &pi))
        return false;

    ULONGLONG started = GetTickCount64();

    for (;;)
    {
        DWORD waitResult = WaitForSingleObject(pi.hProcess, 150);

        if (waitResult == WAIT_OBJECT_0)
            break;

        if (waitResult == WAIT_FAILED || IsCancelRequested() || GetTickCount64() - started > timeoutMs)
        {
            TerminateProcess(pi.hProcess, 1);
            WaitForSingleObject(pi.hProcess, 1000);
            CloseHandle(pi.hThread);
            CloseHandle(pi.hProcess);
            return false;
        }
    }

    DWORD exitCode = 1;
    GetExitCodeProcess(pi.hProcess, &exitCode);

    CloseHandle(pi.hThread);
    CloseHandle(pi.hProcess);

    if (exitCode != 0 || !FileExists(partPath) || IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());

    if (!MoveFileW(partPath.c_str(), targetPath.c_str()))
        return false;

    return true;
}

static bool DownloadWithPowerShellFallback(const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs)
{
    if (IsCancelRequested())
        return false;

    std::wstring partPath = targetPath + L".part";
    DeleteFileW(partPath.c_str());

    std::wstring psUrl = EscapePowerShellSingleQuoted(url);
    std::wstring psPart = EscapePowerShellSingleQuoted(partPath);

    std::wstring script =
        L"$ProgressPreference='SilentlyContinue'; "
        L"[Net.ServicePointManager]::SecurityProtocol=[Net.SecurityProtocolType]::Tls12; "
        L"$ErrorActionPreference='Stop'; "
        L"try { "
        L"$wc=New-Object Net.WebClient; "
        L"$wc.Headers.Add('User-Agent','Mozilla/5.0'); "
        L"$wc.DownloadFile('" + psUrl + L"','" + psPart + L"'); "
        L"exit 0 "
        L"} catch { "
        L"try { "
        L"Invoke-WebRequest -UseBasicParsing -Headers @{'User-Agent'='Mozilla/5.0'} -Uri '" + psUrl + L"' -OutFile '" + psPart + L"' -TimeoutSec 10; "
        L"exit 0 "
        L"} catch { exit 1 } "
        L"}";

    std::wstring powerShellExe = GetSystemToolPath(L"WindowsPowerShell\\v1.0\\powershell.exe");
    std::wstring encoded = Base64EncodePowerShellCommand(script);
    std::wstring command =
        QuoteProcessArgument(powerShellExe) +
        L" -NoProfile -ExecutionPolicy Bypass -EncodedCommand " + encoded;

    STARTUPINFOW si{};
    si.cb = sizeof(si);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

    PROCESS_INFORMATION pi{};

    std::vector<wchar_t> cmd(command.begin(), command.end());
    cmd.push_back(L'\0');

    const wchar_t* appName = FileExists(powerShellExe) ? powerShellExe.c_str() : nullptr;

    if (!CreateProcessW(appName, cmd.data(), nullptr, nullptr, FALSE, CREATE_NO_WINDOW, nullptr, nullptr, &si, &pi))
        return false;

    ULONGLONG started = GetTickCount64();

    for (;;)
    {
        DWORD waitResult = WaitForSingleObject(pi.hProcess, 150);

        if (waitResult == WAIT_OBJECT_0)
            break;

        if (waitResult == WAIT_FAILED || IsCancelRequested() || GetTickCount64() - started > timeoutMs)
        {
            TerminateProcess(pi.hProcess, 1);
            WaitForSingleObject(pi.hProcess, 1000);
            CloseHandle(pi.hThread);
            CloseHandle(pi.hProcess);
            return false;
        }
    }

    DWORD exitCode = 1;
    GetExitCodeProcess(pi.hProcess, &exitCode);

    CloseHandle(pi.hThread);
    CloseHandle(pi.hProcess);

    if (exitCode != 0 || !FileExists(partPath) || IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());

    if (!MoveFileW(partPath.c_str(), targetPath.c_str()))
        return false;

    return true;
}

static bool FindFirstExeRecursive(const std::wstring& folder, std::wstring& outPath)
{
    std::wstring mask = CombinePath(folder, L"*");

    WIN32_FIND_DATAW data{};
    HANDLE hFind = FindFirstFileW(mask.c_str(), &data);

    if (hFind == INVALID_HANDLE_VALUE)
        return false;

    std::vector<std::wstring> subFolders;

    do
    {
        if (wcscmp(data.cFileName, L".") == 0 || wcscmp(data.cFileName, L"..") == 0)
            continue;

        std::wstring path = CombinePath(folder, data.cFileName);

        if (data.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
        {
            subFolders.push_back(path);
        }
        else if (EndsWithI(path, L".exe"))
        {
            outPath = path;
            FindClose(hFind);
            return true;
        }
    }
    while (FindNextFileW(hFind, &data));

    FindClose(hFind);

    for (const auto& subFolder : subFolders)
    {
        if (FindFirstExeRecursive(subFolder, outPath))
            return true;
    }

    return false;
}

static std::wstring GetItemTargetPath(const DownloadItem& item, const std::wstring& baseFolder)
{
    std::wstring tabFolder = CombinePath(baseFolder, GetTabFolderName(item.tab));
    return CombinePath(tabFolder, item.file);
}

static std::wstring GetItemExtractDir(const DownloadItem& item, const std::wstring& baseFolder)
{
    std::wstring tabFolder = CombinePath(baseFolder, GetTabFolderName(item.tab));
    return CombinePath(tabFolder, RemoveExtension(item.file));
}

static std::wstring FindLaunchPathForItem(const DownloadItem& item, const std::wstring& baseFolder)
{
    std::wstring target = GetItemTargetPath(item, baseFolder);

    if (EndsWithI(target, L".zip"))
    {
        std::wstring extractDir = GetItemExtractDir(item, baseFolder);
        std::wstring exePath;

        if (FindFirstExeRecursive(extractDir, exePath))
            return exePath;

        // ZIP сам по себе не открываем как программу: если он уже скачан,
        // одиночный клик сначала распакует архив через ProcessItem().
        return L"";
    }

    if (FileExists(target))
        return target;

    return L"";
}

static bool OpenPath(const std::wstring& path)
{
    if (path.empty())
        return false;

    HINSTANCE result = ShellExecuteW(
        g_hMain,
        L"open",
        path.c_str(),
        nullptr,
        nullptr,
        SW_SHOWNORMAL
    );

    return reinterpret_cast<INT_PTR>(result) > 32;
}

static bool OpenPathElevated(const std::wstring& path)
{
    if (path.empty())
        return false;

    HINSTANCE result = ShellExecuteW(
        g_hMain,
        L"runas",
        path.c_str(),
        nullptr,
        nullptr,
        SW_SHOWNORMAL
    );

    return reinterpret_cast<INT_PTR>(result) > 32;
}

static bool ShouldOpenItemElevated(const DownloadItem& item)
{
    std::wstring name = ToLower(item.name);

    return name.find(L"registryexplorer") != std::wstring::npos ||
        (name.find(L"registry") != std::wstring::npos && name.find(L"explorer") != std::wstring::npos) ||
        name.find(L"recuva") != std::wstring::npos;
}

static bool OpenItemPath(const DownloadItem& item, const std::wstring& path)
{
    if (ShouldOpenItemElevated(item))
        return OpenPathElevated(path);

    return OpenPath(path);
}

static bool CopyTextToClipboard(const wchar_t* text)
{
    if (!text || !g_hMain)
        return false;

    size_t len = wcslen(text);
    HGLOBAL memory = GlobalAlloc(GMEM_MOVEABLE, (len + 1) * sizeof(wchar_t));

    if (!memory)
        return false;

    wchar_t* target = static_cast<wchar_t*>(GlobalLock(memory));

    if (!target)
    {
        GlobalFree(memory);
        return false;
    }

    memcpy(target, text, (len + 1) * sizeof(wchar_t));
    GlobalUnlock(memory);

    if (!OpenClipboard(g_hMain))
    {
        GlobalFree(memory);
        return false;
    }

    EmptyClipboard();
    SetClipboardData(CF_UNICODETEXT, memory);
    CloseClipboard();
    return true;
}

static void CopyHelperText(const wchar_t* text, const std::wstring& name)
{
    if (CopyTextToClipboard(text))
        PostLog(L"Скопировано: " + name);
    else
        PostLog(L"Не скопировано: " + name);
}

static const wchar_t* EVERYTHING_QUERY_1 = LR"HCEV1(ext:.exe;.jar;.zip;.rar regex:(impact|wurst|bleach[-_]?hack|aristois|huzuni|skill[-_]?client|nodus|inertia|ares|sigma|meteor|BetterHitReg|atomic|zamorozka|liquid[-_]?bounce|nurik|nursultan|celestial|calestial|celka|expensive|neverhook|excellent|wexside|wild|minced|deadcode|akrien|future|dreampool|vape|infinity|squad|no[-_]?rules|konas|zeus[-_]?client|rich[-_]?client|ghost[-_]?client|rusher[-_]?hack|thunder[-_]?hack|moon[-_]?hack|winner|nova|exire|doomsday|nightware|ricardo|extazyy|troxill|arbuz|dauntiblyat|rename[-_]?me[-_]?please|edit[-_]?me|takker|faker|xameleon|fuze[-_]?client|wise[-_]?folder|net[-_]?limiter|feather|delta|eclipse|venus|jex|hakari|hush|hach|rogalik|catlavan|haruka|wissend|fluger|sperma|vortex|newcode|astra|britva|bariton|bot|player|freecam|bedrock|hotbar|swap|chest|gumball|tweak|entity|crystal|lowdurabilityswitcher|optimizer|viabackwards|viaforge|viaproxy|hitbox|elytra|through|mob|auto|place|health|inventory|x[-_]?ray|clean[-_]?cut|smart[-_]?moving|save[-_]?searcher|world[-_]?downloader|trade[-_]?finder|chorus[-_]?find|inv[-_]?move|chunk[-_]?copy|seed[-_]?cracker|diamond[-_]?sim|forge[-_]?hax|step[-_]?up|client[-_]?commands|camera[-_]?utils|cheat[-_]?utils|universal[-_]?mod|swing[-_]?through[-_]?grass))HCEV1";

static const wchar_t* EVERYTHING_QUERY_2 = LR"HCEV2(ext:.txt;.json;.toml;.yml;.cfg;.properties;.ini | folder: dm:last14days regex:(shellbag|bariton|bot|player|freecam|bedrock|hotbar|swap|chest|gumball|tweak|entity|crystal|optimizer|viabackwards|viaforge|viaproxy|hitbox|elytra|through|mob|auto|place|health|inventory|x[-_]?ray|clean[-_]?cut|smart[-_]?moving|save[-_]?searcher|world[-_]?downloader|trade[-_]?finder|chorus[-_]?find|inv[-_]?move|chunk[-_]?copy|seed[-_]?cracker|diamond[-_]?sim|forge[-_]?hax|step[-_]?up|client[-_]?commands|camera[-_]?utils|cheat[-_]?utils|universal[-_]?mod|swing[-_]?through[-_]?grass))HCEV2";

// Everything and PowerShell helper text files are no longer created on disk.
// Texts are available inside the utility via copy buttons.


static const wchar_t* JOURNAL_TRACE_TXT = LR"HCJOURNAL(1
directory:.mineceaft;name:!!.
2
directory:.minecraft;directory:config||mods||logs||addons
3
directory::;name:.jar||.zip||.rar;name:!!.lnk
4
directory:Downloads||Desktop
5
directory:prefetch)HCJOURNAL";


static const wchar_t* SYSTEM_INFORMER_TXT = LR"HCSYSTEM(Regex (case-insensetive) | javaw.exe:

(ASM:|H\(q=Zf@wDLF\?/d\?D&|x/fC5K<;i/f6I\-w\[|SMe;8#xj0s<\+pyE&bEZ|cR_A\^yvDVSXDYPQp\]|cortex|r\|\~\}Xnpo\>\]_\^'@BA|WYX\-gih\>rssK\{\|\|\[|net\/minecraft\/client\/renderer\/RenderState\$\$Lambda\+0x000001e7d29d9930|7ba\{vsdkvhd\}o\}w|Hba\{vsdkvhduy\{q|Fba\{vsdkvhdpxhz|Aba\{vsdkvhdnxhv|x\/mo\/c|Self Destruct|blessed|stubborn\.website|cortex|E S P|Hitbox:|Reach:|chs\/Main|chs\/Profiler|net\.minecraftforge\.ASMEventHandler\.31\.wait\(long, int\)|magicthein|radioegor146|BaoBab:|b\/time|chs\/90|by\.kayn|ZDCoder|dreampool|TriggerBOT|forge\.commons\.|sjuuOiqotmus|<!+CJR|GUI Blur)


Regex (case-insensetive) | DPS:

(?:2023\/03\/09:17:11:30|2023\/07\/03:22:01:11|2023\/10\/04:16:07:05|2024\/04\/05:11:43:39|2025\/05\/28:23:57:12|2076\/05\/18:04:53:15|2024\/05\/02:22:48|2024\/09\/06:09:35:50|2023\/08\/13:13:09:58|2025\/01\/20:21:37:53|2025\/01\/16:14:21:21|2022\/06\/07:07:45:55)

> Список dps для справки (НЕ ДЛЯ БАНА!)

2023/03/09:17:11:30 - stubborn
2023/07/03:22:01:11 - синий авалон
2023/10/04:16:07:05 - зелёный авалон
2024/04/05:11:43:39 - красный авалон
2025/05/28:23:57:12 - cortex
2076/05/18:04:53:15 - unicorn
2024/05/02:22:48 - niobium
2024/09/06:09:35:50 - ocean bypass
2023/08/13:13:09:58 - rockstar
2025/01/20:21:37:53 - troxill
2025/01/16:14:21:21 - nemezida
2022/06/07:07:45:55 - mp3)HCSYSTEM";

static const wchar_t* POWERSHELL_TXT = LR"HCPOWERSHELL($ErrorActionPreference="SilentlyContinue";$out="$PWD\check.txt";$twinksOut="$PWD\checktwinks.txt";$mac = (Get-NetAdapter -Physical | Where-Object {$_.Status -eq 'Up'} | Sort-Object -Property InterfaceIndex | Select-Object -First 1).MacAddress;if (-not $mac) { $mac = (Get-NetAdapter | Where-Object {$_.Status -eq 'Up' -and $_.InterfaceDescription -notmatch 'Virtual|WireGuard|TAP'} | Sort-Object -Property InterfaceIndex | Select-Object -First 1).MacAddress };"1. Модель системы: $((Get-WmiObject Win32_ComputerSystem).Model)`n2. Серийный BIOS: $((Get-WmiObject Win32_BIOS).SerialNumber)`n3. Реестр: $((Get-ItemProperty 'HKLM:\HARDWARE\DESCRIPTION\System\BIOS').SystemProductName)`n4. Модель диска: $((Get-WmiObject Win32_DiskDrive | ForEach-Object { "$($_.Model) ($([math]::Round($_.Size/1GB, 2)) GB)" }) -join ', ')`n5. Видеоадаптеры: $(($(Get-CimInstance Win32_PnPSignedDriver | Where-Object {$_.DeviceClass -eq 'DISPLAY'} | ForEach-Object { "$($_.DeviceName) | $($_.Manufacturer) | $($_.DriverProviderName)" }) -join '; '))`n6. Виртуальные устройства: $((Get-WmiObject Win32_PnPEntity | Where-Object {$_.Name -match 'VMware|Virtual|VBox|Hyper-V|QEMU|Xen|KVM|VirtIO|VirtualBox|VMCI|SVGA|VMBus'}).Name -join ', ')`n7. Mac: $mac">$out;$bootTime=(Get-CimInstance Win32_OperatingSystem).LastBootUpTime;function f($t){$p=(' '*((45-$t.Length)/2));"${p}${t}${p}"+(' '*(45-$p.Length*2-$t.Length))};"`n`n=============================================`n$(f 'ИНФОРМАЦИЯ О ВКЛЮЧЕНИИ ПК')`n=============================================`nПоследнее включение ПК: $($bootTime.ToString('yyyy-MM-dd HH:mm:ss'))">>$out;"`n`n`n=============================================`n$(f 'СОСТОЯНИЕ СЛУЖБ')`n=============================================">>$out;$services=@("PcaSvc","DPS","SysMain","EventLog","bam")|%{$s=Get-Service $_ -ea 0;if($s){$t=(Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\$_" -n Start -ea 0).Start;$n=switch($t){2{'Авто'}3{'Вручную'}4{'Отключена'}default{'Неизвестно'}};$p=if($s.Status-eq'Running'){try{$procId=(Get-WmiObject Win32_Service -Filter "Name='$_'").ProcessID;(Get-Process -Id $procId -ea 0).StartTime.ToString('yyyy-MM-dd HH:mm:ss')}catch{'Не удалось получить'}}else{'Не запущена'};[PSCustomObject]@{'Служба'=$_.PadRight(8);'Состояние'=$s.Status.ToString().PadRight(10);'Тип запуска'=$n.PadRight(9);'Время запуска'=$p.PadRight(20)}}};($services|ft -AutoSize -Wrap|Out-String -Width 4096|%{$_.Trim()})>>$out;"`n`n`n=============================================`n$(f 'СОБЫТИЯ ОЧИСТКИ ЖУРНАЛОВ (ID 104)')`n=============================================">>$out;$e=Get-WinEvent -LogName System -FilterXPath "*[System[(EventID=104)]]" -ea 0;if($e){$e|%{$d=($_.Message-split"`n")[0];"ID $($_.Id) | $($_.TimeCreated.ToString('yyyy-MM-dd HH:mm:ss')) | $d">>$out}};"`n`n`n=============================================`n$(f 'СОБЫТИЯ ПРИЛОЖЕНИЯ (ID 3079)')`n=============================================">>$out;Get-WinEvent -LogName Application -FilterXPath "*[System[(EventID=3079)]]" -MaxEvents 5 -ea 0|%{$m=($_.Message-split"Подробности:")[-1].Trim()-replace"`n|`r|\s+"," ";"ID $($_.Id) | $($_.TimeCreated.ToString('yyyy-MM-dd HH:mm:ss')) | $m">>$out};"`n`n`n=============================================`n$(f 'СПИСОК ТВИНКОВ ИГРОКА')`n=============================================">>$out;$r=@{};$err=@();Get-ChildItem -Path . -Recurse -Include *.log,*.log.gz -ea 0|%{try{if($_.Extension-eq'.gz'){$c=[System.IO.Compression.GzipStream]::new([System.IO.File]::OpenRead($_.FullName),[System.IO.Compression.CompressionMode]::Decompress);$s=[System.IO.StreamReader]::new($c);while(-not$s.EndOfStream){$l=$s.ReadLine();if($l-match'setting user:\s*(.+)$'){$r[$matches[1].Trim()]=1}};$s.Close()}else{Get-Content $_.FullName|?{$_-match'setting user:\s*(.+)$'}|%{$r[$matches[1].Trim()]=1}}}catch{$err+="Ошибка логов $($_.Name): $_"}};if(Test-Path "$env:APPDATA\.iasx"){try{[System.Text.Encoding]::UTF8.GetString([IO.File]::ReadAllBytes("$env:APPDATA\.iasx")) -split '[\0-\x1F\x7F-\x9F]'|%{$_.TrimEnd('t')}|?{$_-match '^[\p{IsCyrillic}\p{IsBasicLatin}_]{3,20}$'-and$_-notmatch'=$'-and$_-notmatch'(java|alias|user|pass|field|object|count|list|data|xp|sr|lastused|premium|obj|q$|\[)'}|%{$r[$_]=1}}catch{$err+="Ошибка appdata iasx: $_"}};Get-ChildItem -Path . -Recurse -Filter *.iasx -ea 0|%{try{[System.Text.Encoding]::UTF8.GetString([IO.File]::ReadAllBytes($_.FullName)) -split '[\0-\x1F\x7F-\x9F]'|%{$_.TrimEnd('t')}|?{$_-match '^[\p{IsCyrillic}\p{IsBasicLatin}_]{3,20}$'-and$_-notmatch'=$'-and$_-notmatch'(java|alias|user|pass|field|object|count|list|data|xp|sr|lastused|premium|obj|q$|\[)'}|%{$r[$_]=1}}catch{$err+="Ошибка iasx $($_.Name): $_"}};Get-ChildItem -Path . -Recurse -Filter accounts_v1.do_not_send_to_anyone -ea 0|%{try{$b=[System.IO.File]::ReadAllBytes($_.FullName);$ms=New-Object System.IO.MemoryStream(,$b[2..($b.Length-1)]);$ds=New-Object System.IO.Compression.DeflateStream($ms,[System.IO.Compression.CompressionMode]::Decompress);$sr=New-Object System.IO.StreamReader($ds);$c=$sr.ReadToEnd();$sr.Close();$ds.Close();$ms.Close();$c-split'[\x00-\x1F]'|?{$_-match'^[a-zA-Z0-9_]{3,}$'}|%{$r[$_]=1}}catch{$err+="Ошибка accounts_v1 $($_.Name): $_"}};if(Test-Path "$PWD\usercache.json"){try{Get-Content "$PWD\usercache.json" -Raw|ConvertFrom-Json|Select-Object -ExpandProperty name|%{$r[$_]=1}}catch{$err+="Ошибка usercache.json: $_"}};if(Test-Path "$PWD\TlauncherProfiles.json"){try{Get-Content "$PWD\TlauncherProfiles.json" -Raw|ConvertFrom-Json|%{$_.accounts.PSObject.Properties.Value|Select-Object -Expand username}|%{$r[$_]=1}}catch{$err+="Ошибка TlauncherProfiles.json: $_"}};if(Test-Path "$PWD\LabyMod\accounts.json"){try{Get-Content "$PWD\LabyMod\accounts.json" -Raw|ConvertFrom-Json|%{$_.accounts.PSObject.Properties.Value.username}|%{$r[$_]=1}}catch{$err+="Ошибка LabyMod/accounts.json: $_"}};if(Test-Path "$PWD\labymod-neo\accounts.json"){try{Get-Content "$PWD\labymod-neo\accounts.json" -Raw|ConvertFrom-Json|%{$_.accounts.PSObject.Properties.Value.username}|%{$r[$_]=1}}catch{$err+="Ошибка labymod-neo/accounts.json: $_"}};Get-ChildItem -Path . -Recurse -Filter ias.json -ea 0|%{try{$j=Get-Content $_.FullName -Raw|ConvertFrom-Json;if($j.accounts){$j.accounts|%{if($_.name){$r[$_.name]=1}elseif($_.data.username){$r[$_.data.username]=1}}}}catch{$err+="Ошибка ias.json $($_.Name): $_"}};if(Test-Path "$PWD\usernamecache.json"){try{Get-Content "$PWD\usernamecache.json" -Raw|ConvertFrom-Json|%{$_.PSObject.Properties.Value}|%{$r[$_]=1}}catch{$err+="Ошибка usernamecache.json: $_"}};if(Test-Path "$env:APPDATA\.lunarclient"){try{Get-ChildItem "$env:APPDATA\.lunarclient" -Recurse -Filter *.json -ea 0|%{try{$c=Get-Content $_.FullName -Raw;if($c -match'"username"\s*:\s*"([^"]+)"'){$matches[1]|%{$r[$_]=1}}if($c -match'"displayName"\s*:\s*"([^"]+)"'){$matches[1]|%{$r[$_]=1}}}catch{$err+="Ошибка Lunar Client $($_.Name): $_"}}}catch{$err+="Ошибка доступа к Lunar Client: $_"}};if(Test-Path "$env:APPDATA\lunarclient"){try{Get-ChildItem "$env:APPDATA\lunarclient" -Recurse -Filter *.json -ea 0|%{try{$c=Get-Content $_.FullName -Raw;if($c -match'"username"\s*:\s*"([^"]+)"'){$matches[1]|%{$r[$_]=1}}if($c -match'"displayName"\s*:\s*"([^"]+)"'){$matches[1]|%{$r[$_]=1}}}catch{$err+="Ошибка Lunar Client $($_.Name): $_"}}}catch{$err+="Ошибка доступа к Lunar Client: $_"}};if($r.Count-gt0){($r.Keys-join' ')>$twinksOut;$r.Keys>>$out}else{"Нет данных о твинках">>$out};if($err){"Ошибки:`n$($err-join"`n")">>$out}
)HCPOWERSHELL";

static const wchar_t* POWERSHELL_SITES_TXT = LR"HCSITES($sites="anticheat.ac","mods.holyworld.me","github.com","voidtools.com","privazer.com","nirsoft.net","sourceforge.net","download.ericzimmermanstools.com","win-rar.com","simpleunlocker.ds1nc.ru"; function Test-Site($u){ $r=@(); if(Select-String -Path "$env:SystemRoot\System32\drivers\etc\hosts" -Pattern "(?mi)^\s*?\d.+?$($u.Replace('.','\.'))\s*$"){$r+="HOSTS"}; try{if((Resolve-DnsName $u -ErrorAction Stop -Type A|% IPAddress)-like'127.*'){$r+="DNS_LOOPBACK"}}catch{}; $regPath="HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\$($u.Replace('.','\\'))";if(Test-Path $regPath){$r+="REGISTRY_ZONE"}elseif(Test-Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\$u"){$r+="REGISTRY_ZONE"}; if(Get-NetFirewallRule|?{$_.DisplayName-like"*$u*"-and$_.Action-eq"Block"}){$r+="FIREWALL"}; try{iwr "https://$u"-TimeoutSec 5 -UseBasicParsing -DisableKeepAlive -ErrorAction Stop|Out-Null}catch{if($_.Exception.Response.StatusCode-eq407){$r+="PROXY_BLOCK"}elseif(-not$_.Exception.Response){try{iwr "http://$u"-TimeoutSec 3 -UseBasicParsing -DisableKeepAlive -ErrorAction Stop|Out-Null}catch{$r+="WEB_BLOCKED"}}}; $tcp80=Test-NetConnection $u -Port 80 -WarningAction SilentlyContinue; $tcp443=Test-NetConnection $u -Port 443 -WarningAction SilentlyContinue; if(-not$tcp80.TcpTestSucceeded-and$u-ne'anticheat.ac'){$r+="PORT_80_BLOCKED"};if(-not$tcp443.TcpTestSucceeded-and$u-ne'anticheat.ac'){$r+="PORT_443_BLOCKED"}; if($r.Count-gt0){"$u → БЛОКИРОВКА: $($r-join', ')"}else{"$u → OK"}}; $sites|%{Test-Site $_}
)HCSITES";


static bool WriteUtf8TextFile(const std::wstring& path, const wchar_t* text)
{
    if (!text)
        return false;

    int size = WideCharToMultiByte(CP_UTF8, 0, text, -1, nullptr, 0, nullptr, nullptr);

    if (size <= 1)
        return false;

    std::string utf8(size, '\0');
    WideCharToMultiByte(CP_UTF8, 0, text, -1, &utf8[0], size, nullptr, nullptr);

    HANDLE file = CreateFileW(
        path.c_str(),
        GENERIC_WRITE,
        0,
        nullptr,
        CREATE_ALWAYS,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
        return false;

    const BYTE bom[] = { 0xEF, 0xBB, 0xBF };
    DWORD written = 0;
    bool ok = WriteFile(file, bom, sizeof(bom), &written, nullptr) && written == sizeof(bom);

    if (ok)
    {
        written = 0;
        ok = WriteFile(file, utf8.data(), static_cast<DWORD>(utf8.size() - 1), &written, nullptr) &&
            written == static_cast<DWORD>(utf8.size() - 1);
    }

    CloseHandle(file);
    return ok;
}

static bool ItemNameEquals(const DownloadItem& item, const std::wstring& name)
{
    return ToLower(item.name) == ToLower(name);
}

static bool ReadFilePrefix(const std::wstring& path, std::vector<unsigned char>& out, DWORD maxBytes = 16)
{
    out.clear();

    HANDLE file = CreateFileW(
        path.c_str(),
        GENERIC_READ,
        FILE_SHARE_READ,
        nullptr,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
        return false;

    out.resize(maxBytes);
    DWORD read = 0;
    bool ok = ReadFile(file, out.data(), maxBytes, &read, nullptr) != FALSE;
    CloseHandle(file);

    if (!ok || read == 0)
    {
        out.clear();
        return false;
    }

    out.resize(read);
    return true;
}

static bool StartsWithBytes(const std::vector<unsigned char>& data, std::initializer_list<unsigned char> bytes)
{
    if (data.size() < bytes.size())
        return false;

    size_t i = 0;
    for (unsigned char b : bytes)
    {
        if (data[i++] != b)
            return false;
    }

    return true;
}

static bool LooksLikeDownloadedHtml(const std::vector<unsigned char>& data)
{
    if (data.empty())
        return false;

    std::string head;
    for (unsigned char c : data)
    {
        if (c >= 32 && c <= 126)
            head.push_back(static_cast<char>(tolower(c)));
    }

    return head.find("html") != std::string::npos ||
        head.find("doctype") != std::string::npos ||
        head.find("<head") != std::string::npos ||
        head.find("<body") != std::string::npos;
}


static bool IsZipArchiveStructurallyValid(const std::wstring& path)
{
    unsigned long long totalSize = 0;
    if (!GetFileSizeBytes(path, totalSize) || totalSize < 22)
        return false;

    HANDLE file = CreateFileW(
        path.c_str(),
        GENERIC_READ,
        FILE_SHARE_READ,
        nullptr,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
        return false;

    DWORD tailSize = static_cast<DWORD>(std::min<unsigned long long>(totalSize, 65557));
    LARGE_INTEGER offset{};
    offset.QuadPart = static_cast<LONGLONG>(totalSize - tailSize);

    if (!SetFilePointerEx(file, offset, nullptr, FILE_BEGIN))
    {
        CloseHandle(file);
        return false;
    }

    std::vector<unsigned char> tail(tailSize);
    DWORD read = 0;
    bool ok = ReadFile(file, tail.data(), tailSize, &read, nullptr) != FALSE;
    CloseHandle(file);

    if (!ok || read < 22)
        return false;

    tail.resize(read);

    // End of Central Directory signature. This catches most truncated ZIP/JAR files
    // without extracting the archive.
    for (size_t i = tail.size() - 22; i != static_cast<size_t>(-1); --i)
    {
        if (i + 4 <= tail.size() &&
            tail[i] == 0x50 && tail[i + 1] == 0x4B && tail[i + 2] == 0x05 && tail[i + 3] == 0x06)
        {
            return true;
        }

        if (i == 0)
            break;
    }

    // Some ZIP64 archives can be unusual; still require a central-directory marker.
    for (size_t i = 0; i + 4 <= tail.size(); ++i)
    {
        if (tail[i] == 0x50 && tail[i + 1] == 0x4B &&
            ((tail[i + 2] == 0x06 && tail[i + 3] == 0x06) ||
             (tail[i + 2] == 0x06 && tail[i + 3] == 0x07)))
        {
            return true;
        }
    }

    return false;
}

static bool IsValidDownloadedPayload(const DownloadItem& item, const std::wstring& path)
{
    if (!FileExists(path))
        return false;

    LARGE_INTEGER size{};
    HANDLE file = CreateFileW(
        path.c_str(),
        GENERIC_READ,
        FILE_SHARE_READ,
        nullptr,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        nullptr
    );

    if (file == INVALID_HANDLE_VALUE)
        return false;

    bool gotSize = GetFileSizeEx(file, &size) != FALSE;
    CloseHandle(file);

    if (!gotSize || size.QuadPart <= 0)
        return false;

    std::wstring fileLowerForSize = ToLower(item.file);

    if (EndsWithI(fileLowerForSize, L".exe") && size.QuadPart < 4096)
        return false;

    if ((EndsWithI(fileLowerForSize, L".zip") || EndsWithI(fileLowerForSize, L".jar")) && size.QuadPart < 22)
        return false;

    // Do not hardcode a large minimum size for Eric Zimmerman GUI tools.
    // Their package sizes can change between releases. A valid ZIP signature
    // plus non-HTML payload is a better generic validation here; a too-large
    // minimum caused real downloads to be deleted as "invalid".

    if (EndsWithI(fileLowerForSize, L".rar") && size.QuadPart < 7)
        return false;

    if (EndsWithI(fileLowerForSize, L".7z") && size.QuadPart < 6)
        return false;

    if (EndsWithI(fileLowerForSize, L".msi") && size.QuadPart < 512)
        return false;

    std::vector<unsigned char> head;
    if (!ReadFilePrefix(path, head, 32))
        return false;

    if (LooksLikeDownloadedHtml(head))
        return false;

    std::wstring fileLower = fileLowerForSize;

    if (EndsWithI(fileLower, L".exe"))
        return StartsWithBytes(head, { 0x4D, 0x5A });

    if (EndsWithI(fileLower, L".zip") || EndsWithI(fileLower, L".jar"))
        return StartsWithBytes(head, { 0x50, 0x4B }) && IsZipArchiveStructurallyValid(path);

    if (EndsWithI(fileLower, L".rar"))
        return StartsWithBytes(head, { 0x52, 0x61, 0x72, 0x21 });

    if (EndsWithI(fileLower, L".7z"))
        return StartsWithBytes(head, { 0x37, 0x7A, 0xBC, 0xAF, 0x27, 0x1C });

    if (EndsWithI(fileLower, L".msi"))
        return StartsWithBytes(head, { 0xD0, 0xCF, 0x11, 0xE0 });

    return true;
}

static bool ValidateDownloadedFileOrDelete(const DownloadItem& item, const std::wstring& targetPath)
{
    if (IsValidDownloadedPayload(item, targetPath))
        return true;

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());
    PostLog(L"Некорректный файл: " + item.name);
    return false;
}

static bool DownloadRecursiveValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath)
{
    if (!DownloadRecursive(url, targetPath, item.name, 0))
        return false;

    return ValidateDownloadedFileOrDelete(item, targetPath);
}

static bool DownloadCurlValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath)
{
    if (!DownloadWithCurlFallback(url, targetPath, GetPowerShellTimeoutMs(item)))
        return false;

    return ValidateDownloadedFileOrDelete(item, targetPath);
}

static bool DownloadPowerShellValidated(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath)
{
    if (!DownloadWithPowerShellFallback(url, targetPath, GetPowerShellTimeoutMs(item)))
        return false;

    return ValidateDownloadedFileOrDelete(item, targetPath);
}

static bool IsPrimaryUrlOnlyItem(const DownloadItem& item)
{
    return IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item) || IsSimpleUnlockerItem(item);
}

static DWORD GetPrimaryUrlOnlyTimeoutMs(const DownloadItem& item)
{
    if (IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item))
        return 180000;

    if (IsSimpleUnlockerItem(item))
        return 60000;

    return GetPowerShellTimeoutMs(item);
}

static bool DownloadPrimaryUrlOnlyValidated(const DownloadItem& item, const std::wstring& targetPath)
{
    if (IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());

    DWORD timeoutMs = GetPrimaryUrlOnlyTimeoutMs(item);

    PostLog(L"Прямая ссылка: " + item.name);

    if (IsRegistryExplorerItem(item) || IsJumpListExplorerItem(item))
    {
        if (DownloadWithCurlBrowserLike(item.url, targetPath, timeoutMs) &&
            ValidateDownloadedFileOrDelete(item, targetPath))
        {
            return true;
        }
    }
    else
    {
        if (DownloadWithCurlFallback(item.url, targetPath, timeoutMs) &&
            ValidateDownloadedFileOrDelete(item, targetPath))
        {
            return true;
        }
    }

    if (IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());

    if (DownloadWithPowerShellFallback(item.url, targetPath, timeoutMs) &&
        ValidateDownloadedFileOrDelete(item, targetPath))
    {
        return true;
    }

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());
    return false;
}


static void AddEricZimmermanKnownUrls(const DownloadItem& item, std::vector<std::wstring>& urls)
{
    if (IsRegistryExplorerItem(item))
    {
        AddUniqueString(urls, L"https://download.ericzimmermanstools.com/net9/RegistryExplorer.zip");
        AddUniqueString(urls, L"https://f001.backblazeb2.com/file/EricZimmermanTools/net9/RegistryExplorer.zip");
        AddUniqueString(urls, L"https://f001.backblazeb2.com/file/EricZimmermanTools/RegistryExplorer.zip");
    }
    else if (IsJumpListExplorerItem(item))
    {
        AddUniqueString(urls, L"https://download.ericzimmermanstools.com/net9/JumpListExplorer.zip");
        AddUniqueString(urls, L"https://f001.backblazeb2.com/file/EricZimmermanTools/net9/JumpListExplorer.zip");
        AddUniqueString(urls, L"https://f001.backblazeb2.com/file/EricZimmermanTools/JumpListExplorer.zip");
    }
}

static std::vector<std::wstring> GetEricZimmermanInstallUrls(const DownloadItem& item)
{
    std::vector<std::wstring> urls;

    // First try to discover the current official link from the live index page.
    // The file names and versions may change; the official page is the source of
    // truth and also avoids stale hardcoded direct URLs.
    std::vector<unsigned char> html;
    const std::wstring home = L"https://ericzimmerman.github.io/";

    if (FetchUrlBytes(home, item.name, html))
    {
        std::wstring discovered = FindBestDownloadLinkForItem(home, html, item);
        if (!discovered.empty())
            AddUniqueString(urls, discovered);
    }

    AddEricZimmermanKnownUrls(item, urls);
    return urls;
}

static bool TryInstallDownloadUrl(const DownloadItem& item, const std::wstring& url, const std::wstring& targetPath, DWORD timeoutMs)
{
    if (IsCancelRequested())
        return false;

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());

    // 1) Native WinHTTP. This keeps the utility independent of curl/PowerShell.
    if (DownloadRecursiveValidated(item, url, targetPath))
        return true;

    if (IsCancelRequested())
        return false;

    // 2) Windows curl. This behaves closest to a manual browser/CDN download.
    if (DownloadWithCurlBrowserLike(url, targetPath, timeoutMs) &&
        ValidateDownloadedFileOrDelete(item, targetPath))
    {
        return true;
    }

    if (IsCancelRequested())
        return false;

    // 3) PowerShell fallback. This is slower, but useful on systems where WinHTTP
    // or curl is blocked/missing. It is intentionally kept last.
    if (DownloadWithPowerShellFallback(url, targetPath, timeoutMs) &&
        ValidateDownloadedFileOrDelete(item, targetPath))
    {
        return true;
    }

    DeleteFileW(targetPath.c_str());
    DeleteFileW((targetPath + L".part").c_str());
    return false;
}

static bool DownloadEricZimmermanLargeValidated(const DownloadItem& item, const std::wstring& targetPath)
{
    std::lock_guard<std::mutex> lock(g_ericDownloadMutex);

    const DWORD perUrlTimeoutMs = 900000;
    std::vector<std::wstring> urls = GetEricZimmermanInstallUrls(item);

    if (urls.empty())
    {
        PostLog(L"Eric: нет ссылок " + item.name);
        return false;
    }

    for (size_t i = 0; i < urls.size(); ++i)
    {
        if (IsCancelRequested())
            return false;

        PostLog(L"Eric " + std::to_wstring(i + 1) + L"/" + std::to_wstring(urls.size()) + L": " + item.name);

        if (TryInstallDownloadUrl(item, urls[i], targetPath, perUrlTimeoutMs))
            return true;
    }

    PostLog(L"Eric: установка не удалась " + item.name);
    return false;
}

static void CleanupOldJuniorTextFiles(const std::wstring& tabFolder)
{
    // Helper text files are stored in the UI now, so remove leftovers from older builds.
    DeleteFileW(CombinePath(tabFolder, L"everything.txt").c_str());
    DeleteFileW(CombinePath(tabFolder, L"journaltrace.txt").c_str());
    DeleteFileW(CombinePath(tabFolder, L"systeminformer.txt").c_str());
}

static size_t DeleteXmlFilesRecursive(const std::wstring& folder)
{
    if (!DirectoryExists(folder))
        return 0;

    size_t deletedCount = 0;
    std::wstring mask = CombinePath(folder, L"*");

    WIN32_FIND_DATAW data{};
    HANDLE hFind = FindFirstFileW(mask.c_str(), &data);

    if (hFind == INVALID_HANDLE_VALUE)
        return 0;

    do
    {
        if (wcscmp(data.cFileName, L".") == 0 || wcscmp(data.cFileName, L"..") == 0)
            continue;

        std::wstring path = CombinePath(folder, data.cFileName);

        if (data.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
        {
            deletedCount += DeleteXmlFilesRecursive(path);
        }
        else if (EndsWithI(path, L".xml"))
        {
            if (DeleteFileW(path.c_str()))
                ++deletedCount;
        }
    }
    while (FindNextFileW(hFind, &data));

    FindClose(hFind);
    return deletedCount;
}

static void CleanupXmlFiles(const std::wstring& folder)
{
    size_t deletedCount = DeleteXmlFilesRecursive(folder);

    if (deletedCount > 0)
        PostLog(L"XML удалены: " + std::to_wstring(deletedCount));
}
static bool WaitForExtractedExe(const std::wstring& extractDir, std::wstring& exePath, DWORD timeoutMs = ZIP_EXTRACT_TIMEOUT_MS)
{
    ULONGLONG started = GetTickCount64();

    do
    {
        if (IsCancelRequested())
            return false;

        if (FindFirstExeRecursive(extractDir, exePath))
            return true;

        Sleep(ZIP_EXTRACT_POLL_MS);
    }
    while (GetTickCount64() - started < timeoutMs);

    return false;
}


static bool ShouldExtractOnlyOnClick(const DownloadItem& item)
{
    (void)item;
    return false;
}

static bool ProcessItem(const DownloadItem& item, const std::wstring& baseFolder, bool extractZip = true)
{
    if (IsCancelRequested())
        return false;

    std::wstring tabFolder = CombinePath(baseFolder, GetTabFolderName(item.tab));

    if (!EnsureDirectory(tabFolder))
    {
        PostLog(L"Не создана папка: " + GetTabFolderName(item.tab));
        return false;
    }

    if (item.tab == 0)
        CleanupOldJuniorTextFiles(tabFolder);

    std::wstring target = CombinePath(tabFolder, item.file);
    bool hadFileBefore = FileExists(target);

    if (hadFileBefore && !ValidateDownloadedFileOrDelete(item, target))
    {
        hadFileBefore = false;
        PostLog(L"Перекачка: " + item.name);
    }

    if (!hadFileBefore)
    {
        PostLog(L"Скачивается: " + item.name);
        AddActiveDownload(item.name);

        bool downloaded = false;

        if (IsPrimaryUrlOnlyItem(item))
        {
            downloaded = DownloadPrimaryUrlOnlyValidated(item, target);
        }
        else
        {
            if (!downloaded && !IsCancelRequested())
                downloaded = DownloadRecursiveValidated(item, item.url, target);

            if (!downloaded && !IsCancelRequested())
                downloaded = DownloadCurlValidated(item, item.url, target);

            if (!downloaded && !IsCancelRequested())
                downloaded = DownloadPowerShellValidated(item, item.url, target);

            if (!downloaded && !IsCancelRequested())
                downloaded = TryFastFallbackUrls(item, target);

            if (!downloaded && !IsCancelRequested() && !IsFastFailItem(item))
                downloaded = TrySoftPortalFallback(item, target);
        }

        RemoveActiveDownload(item.name);

        if (!downloaded)
        {
            if (IsCancelRequested())
                PostLog(L"Отменено: " + item.name);
            else
                PostLog(L"Не скачалось: " + item.name);

            return false;
        }

        PostLog(L"Скачано: " + item.name);
    }

    if (IsCancelRequested())
        return false;

    if (extractZip && FileExists(target) && EndsWithI(target, L".zip"))
    {
        std::wstring extractDir = CombinePath(tabFolder, RemoveExtension(item.file));
        std::wstring exePath;

        if (!FindFirstExeRecursive(extractDir, exePath))
        {
            PostLog(L"Распаковка: " + item.name);

            if (!ExtractZip(target, extractDir))
            {
                PostLog(L"Ошибка распаковки: " + item.name);
                return false;
            }

            CleanupXmlFiles(extractDir);
        }
        else
        {
            CleanupXmlFiles(extractDir);
        }

        if (!WaitForExtractedExe(extractDir, exePath))
        {
            PostLog(L"EXE не найден после распаковки: " + item.name);
            return false;
        }
    }

    CleanupXmlFiles(tabFolder);

    return FileExists(target) && !IsCancelRequested();
}

static void DownloadWorker(std::vector<DownloadItem> items)
{
    g_working = true;

    std::stable_sort(items.begin(), items.end(), [](const DownloadItem& a, const DownloadItem& b)
    {
        return !IsFastFailItem(a) && IsFastFailItem(b);
    });

    std::wstring folder = GetHolyCheckFolder();

    if (!EnsureDirectory(folder))
    {
        PostLog(L"Не создана папка holycheck");
        PostProgress(0);

        if (g_hMain && !g_destroying.load())
            PostMessageW(g_hMain, WM_APP_DONE, 0, 0);
        else
            g_working = false;

        return;
    }

    for (const auto& item : items)
    {
        if (!EnsureDirectory(CombinePath(folder, GetTabFolderName(item.tab))))
            PostLog(L"Не создана папка: " + GetTabFolderName(item.tab));
    }

    std::atomic<int> index = 0;
    std::atomic<int> done = 0;

    int total = static_cast<int>(items.size());
    int workers = static_cast<int>(std::thread::hardware_concurrency());

    if (workers < 2) workers = 2;
    if (workers > 4) workers = 4;
    if (workers > total) workers = total;
    if (workers < 1) workers = 1;

    std::vector<std::thread> threads;

    for (int i = 0; i < workers; ++i)
    {
        threads.emplace_back([&]()
        {
            for (;;)
            {
                int n = index.fetch_add(1);

                if (n >= total || IsCancelRequested())
                    break;

                bool extractNow = !ShouldExtractOnlyOnClick(items[n]);
                ProcessItem(items[n], folder, extractNow);

                int currentDone = done.fetch_add(1) + 1;
                int percent = total > 0 ? currentDone * 100 / total : 100;

                PostProgress(percent);
            }
        });
    }

    for (auto& t : threads)
    {
        if (t.joinable())
            t.join();
    }

    CleanupXmlFiles(folder);

    ClearActiveDownloads();
    InvalidateBottomPanel();

    if (IsCancelRequested())
    {
        PostLog(L"Загрузки отменены");
        PostProgress(0);
    }
    else
    {
        PostLog(L"Готово");
        PostProgress(100);
    }

    HWND hwnd = g_hMain;

    if (hwnd && !g_destroying.load())
        PostMessageW(hwnd, WM_APP_DONE, 0, 0);
    else
        g_working = false;
}

static void SetDownloadControlsEnabled(bool enabled)
{
    if (g_hDownloadAll)
    {
        EnableWindow(g_hDownloadAll, enabled ? TRUE : FALSE);
        InvalidateRect(g_hDownloadAll, nullptr, FALSE);
    }

    if (g_hDeleteHolyCheck)
    {
        EnableWindow(g_hDeleteHolyCheck, enabled ? TRUE : FALSE);
        InvalidateRect(g_hDeleteHolyCheck, nullptr, FALSE);
    }

    if (g_hCancelDownload)
        InvalidateRect(g_hCancelDownload, nullptr, FALSE);

    for (HWND button : g_tileButtons)
    {
        EnableWindow(button, enabled ? TRUE : FALSE);
        InvalidateRect(button, nullptr, FALSE);
    }
}

static void StartDownload(const std::vector<DownloadItem>& items)
{
    if (items.empty())
        return;

    if (g_working)
        return;

    if (g_activeSingleDownloads.load() > 0)
    {
        PostLog(L"Сейчас идут одиночные загрузки");
        return;
    }

    g_cancelRequested = false;
    AbortActiveHttpHandles();
    g_working = true;
    SetDownloadControlsEnabled(false);
    g_progressPercent = 0;
    ClearDownloadLog();
    ClearActiveDownloads();
    PostLog(L"Старт: " + std::to_wstring(items.size()) + L" шт.");
    InvalidateBottomPanel();

    std::thread(DownloadWorker, items).detach();
}

static void DownloadCurrentTab()
{
    std::vector<DownloadItem> selected;

    for (const auto& item : g_items)
    {
        if (item.tab == g_currentTab)
            selected.push_back(item);
    }

    StartDownload(selected);
}

static void OpenOrDownloadSingleWorker(int itemIndex, DownloadItem item)
{
    std::wstring folder = GetHolyCheckFolder();
    bool foldersOk = EnsureDirectory(folder);

    std::wstring tabFolder = CombinePath(folder, GetTabFolderName(item.tab));
    foldersOk = foldersOk && EnsureDirectory(tabFolder);

    bool processed = foldersOk && ProcessItem(item, folder, true);

    if (processed && !IsCancelRequested())
    {
        std::wstring launchPath = FindLaunchPathForItem(item, folder);

        if (OpenItemPath(item, launchPath))
        {
            PostLog(L"Открыто: " + item.name);
        }
        else
        {
            PostLog(L"Не открылось: " + item.name);
        }
    }
    else if (IsCancelRequested())
    {
        PostLog(L"Отменено: " + item.name);
    }
    else
    {
        PostLog(L"Не удалось: " + item.name);
    }

    int doneNow = g_singleDone.fetch_add(1) + 1;
    int totalNow = std::max(g_singleTotal.load(), doneNow);
    int percent = totalNow > 0 ? doneNow * 100 / totalNow : 100;
    PostProgress(percent);

    if (g_hMain && !g_destroying.load())
        PostMessageW(g_hMain, WM_APP_ITEM_DONE, static_cast<WPARAM>(itemIndex), 0);

    if (g_activeSingleDownloads.fetch_sub(1) == 1)
    {
        ClearActiveDownloads();
        InvalidateBottomPanel();

        if (IsCancelRequested())
        {
            PostProgress(0);
            PostLog(L"Загрузки отменены");
        }
        else
        {
            PostProgress(100);
            PostLog(L"Готово");
        }

        g_cancelRequested = false;
    }
}

static void OpenOrDownloadSingleById(int id)
{
    int index = id - IDC_TILE_BASE;

    if (index < 0 || index >= static_cast<int>(g_items.size()))
        return;

    if (g_working || IsCancelRequested() || IsItemBusyIndex(index))
        return;

    const DownloadItem& item = g_items[index];
    std::wstring folder = GetHolyCheckFolder();
    std::wstring tabFolder = CombinePath(folder, GetTabFolderName(item.tab));

    if (!EnsureDirectory(tabFolder))
    {
        PostLog(L"Не создана папка: " + GetTabFolderName(item.tab));
        return;
    }

    std::wstring quickLaunchPath = FindLaunchPathForItem(item, folder);

    if (!quickLaunchPath.empty() && FileExists(quickLaunchPath))
    {
        if (item.tab == 0)
            CleanupOldJuniorTextFiles(tabFolder);

        OpenItemPath(item, quickLaunchPath);
        return;
    }

    int previousActive = g_activeSingleDownloads.fetch_add(1);

    if (previousActive == 0)
    {
        g_cancelRequested = false;
        AbortActiveHttpHandles();
        g_singleTotal = 0;
        g_singleDone = 0;
        g_progressPercent = 0;
        ClearDownloadLog();
        ClearActiveDownloads();
    }

    g_singleTotal.fetch_add(1);
    SetItemBusyIndex(index, true);
    SetTileEnabledByIndex(index, false);
    PostLog(L"Старт: " + item.name);
    InvalidateBottomPanel();

    std::thread(OpenOrDownloadSingleWorker, index, item).detach();
}

static void CloseProcessesFromFolder(const std::wstring& folder);

static bool DeleteFileOrFolderPermanent(const std::wstring& path)
{
    DWORD attr = GetFileAttributesW(path.c_str());

    if (attr == INVALID_FILE_ATTRIBUTES)
        return true;

    if (attr & (FILE_ATTRIBUTE_READONLY | FILE_ATTRIBUTE_HIDDEN | FILE_ATTRIBUTE_SYSTEM))
    {
        DWORD cleanAttr = attr & ~(FILE_ATTRIBUTE_READONLY | FILE_ATTRIBUTE_HIDDEN | FILE_ATTRIBUTE_SYSTEM);

        if (cleanAttr == 0)
            cleanAttr = FILE_ATTRIBUTE_NORMAL;

        SetFileAttributesW(path.c_str(), cleanAttr);
    }

    if (!(attr & FILE_ATTRIBUTE_DIRECTORY))
        return DeleteFileW(path.c_str()) || GetFileAttributesW(path.c_str()) == INVALID_FILE_ATTRIBUTES;

    std::wstring mask = CombinePath(path, L"*");

    WIN32_FIND_DATAW data{};
    HANDLE hFind = FindFirstFileW(mask.c_str(), &data);

    if (hFind != INVALID_HANDLE_VALUE)
    {
        do
        {
            if (wcscmp(data.cFileName, L".") == 0 || wcscmp(data.cFileName, L"..") == 0)
                continue;

            DeleteFileOrFolderPermanent(CombinePath(path, data.cFileName));
        }
        while (FindNextFileW(hFind, &data));

        FindClose(hFind);
    }

    SetFileAttributesW(path.c_str(), FILE_ATTRIBUTE_NORMAL);
    return RemoveDirectoryW(path.c_str()) || GetFileAttributesW(path.c_str()) == INVALID_FILE_ATTRIBUTES;
}

static bool DeleteDirectoryTree(const std::wstring& folder)
{
    if (!IsSafeHolyCheckDeletionTarget(folder))
    {
        PostLog(L"Небезопасный путь удаления");
        return false;
    }

    if (!DirectoryExists(folder))
        return true;

    for (int attempt = 0; attempt < 6; ++attempt)
    {
        CloseProcessesFromFolder(folder);

        if (DeleteFileOrFolderPermanent(folder))
            return true;

        Sleep(450);
    }

    return !DirectoryExists(folder);
}

static bool RegistryGetString(HKEY key, const wchar_t* valueName, std::wstring& out)
{
    DWORD type = 0;
    DWORD cb = 0;

    if (RegQueryValueExW(key, valueName, nullptr, &type, nullptr, &cb) != ERROR_SUCCESS)
        return false;

    if ((type != REG_SZ && type != REG_EXPAND_SZ) || cb == 0)
        return false;

    std::wstring value(cb / sizeof(wchar_t), L'\0');

    if (RegQueryValueExW(key, valueName, nullptr, &type, reinterpret_cast<LPBYTE>(&value[0]), &cb) != ERROR_SUCCESS)
        return false;

    while (!value.empty() && value.back() == L'\0')
        value.pop_back();

    if (type == REG_EXPAND_SZ)
    {
        wchar_t expanded[4096]{};

        if (ExpandEnvironmentStringsW(value.c_str(), expanded, ARRAYSIZE(expanded)) > 0)
            value = expanded;
    }

    out = value;
    return !out.empty();
}

static bool RunElevatedCommandAndWait(const std::wstring& command)
{
    if (command.empty())
        return false;

    std::wstring params = L"/c " + command;

    SHELLEXECUTEINFOW sei{};
    sei.cbSize = sizeof(sei);
    sei.fMask = SEE_MASK_NOCLOSEPROCESS;
    sei.lpVerb = L"runas";
    sei.lpFile = L"cmd.exe";
    sei.lpParameters = params.c_str();
    sei.nShow = SW_HIDE;

    if (!ShellExecuteExW(&sei))
        return false;

    bool ok = true;

    if (sei.hProcess)
    {
        DWORD waitResult = WaitForSingleObject(sei.hProcess, 120000);
        DWORD exitCode = 1;

        if (waitResult != WAIT_OBJECT_0 || !GetExitCodeProcess(sei.hProcess, &exitCode) || exitCode != 0)
            ok = false;

        CloseHandle(sei.hProcess);
    }

    return ok;
}

static std::wstring MakeUninstallCommand(std::wstring command, bool addNsisSilent)
{
    std::wstring lower = ToLower(command);

    if (lower.find(L"msiexec") != std::wstring::npos)
    {
        size_t pos = lower.find(L"/i");

        if (pos != std::wstring::npos)
            command.replace(pos, 2, L"/X");

        pos = ToLower(command).find(L"-i");

        if (pos != std::wstring::npos)
            command.replace(pos, 2, L"/X");

        if (ToLower(command).find(L"/qn") == std::wstring::npos)
            command += L" /qn";

        if (ToLower(command).find(L"/norestart") == std::wstring::npos)
            command += L" /norestart";
    }
    else if (addNsisSilent)
    {
        std::wstring updatedLower = ToLower(command);

        if (updatedLower.find(L" /s") == std::wstring::npos &&
            updatedLower.find(L" -s") == std::wstring::npos &&
            updatedLower.find(L"/silent") == std::wstring::npos &&
            updatedLower.find(L"/verysilent") == std::wstring::npos &&
            updatedLower.find(L"/quiet") == std::wstring::npos)
        {
            command += L" /S";
        }
    }

    return command;
}

static bool DisplayNameMatchesAny(const std::wstring& displayName, const std::vector<std::wstring>& needles)
{
    std::wstring lowerName = ToLower(displayName);

    for (const auto& needle : needles)
    {
        if (lowerName.find(ToLower(needle)) != std::wstring::npos)
            return true;
    }

    return false;
}

static bool FindUninstallCommandInRoot(
    HKEY root,
    REGSAM viewFlags,
    const std::vector<std::wstring>& displayNameNeedles,
    bool addNsisSilent,
    std::wstring& out)
{
    const wchar_t* uninstallPath = L"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall";

    HKEY hUninstall = nullptr;

    if (RegOpenKeyExW(root, uninstallPath, 0, KEY_READ | viewFlags, &hUninstall) != ERROR_SUCCESS)
        return false;

    bool found = false;

    for (DWORD i = 0; !found; ++i)
    {
        wchar_t subKeyName[512]{};
        DWORD subKeyLen = ARRAYSIZE(subKeyName);

        if (RegEnumKeyExW(hUninstall, i, subKeyName, &subKeyLen, nullptr, nullptr, nullptr, nullptr) != ERROR_SUCCESS)
            break;

        HKEY hApp = nullptr;

        if (RegOpenKeyExW(hUninstall, subKeyName, 0, KEY_READ | viewFlags, &hApp) != ERROR_SUCCESS)
            continue;

        std::wstring displayName;

        if (RegistryGetString(hApp, L"DisplayName", displayName) &&
            DisplayNameMatchesAny(displayName, displayNameNeedles))
        {
            std::wstring quietUninstall;
            std::wstring uninstall;

            if (RegistryGetString(hApp, L"QuietUninstallString", quietUninstall))
                out = MakeUninstallCommand(quietUninstall, addNsisSilent);
            else if (RegistryGetString(hApp, L"UninstallString", uninstall))
                out = MakeUninstallCommand(uninstall, addNsisSilent);

            found = !out.empty();
        }

        RegCloseKey(hApp);
    }

    RegCloseKey(hUninstall);
    return found;
}

static bool FindUninstallCommand(
    const std::vector<std::wstring>& displayNameNeedles,
    bool addNsisSilent,
    std::wstring& out)
{
    return
        FindUninstallCommandInRoot(HKEY_LOCAL_MACHINE, KEY_WOW64_64KEY, displayNameNeedles, addNsisSilent, out) ||
        FindUninstallCommandInRoot(HKEY_LOCAL_MACHINE, KEY_WOW64_32KEY, displayNameNeedles, addNsisSilent, out) ||
        FindUninstallCommandInRoot(HKEY_CURRENT_USER, 0, displayNameNeedles, addNsisSilent, out);
}

static bool UninstallProgramIfPresent(
    const std::wstring& logName,
    const std::vector<std::wstring>& displayNameNeedles,
    bool addNsisSilent)
{
    std::wstring command;

    if (!FindUninstallCommand(displayNameNeedles, addNsisSilent, command))
    {
        PostLog(logName + L": не установлен");
        return true;
    }

    PostLog(logName + L": удаление");

    if (!RunElevatedCommandAndWait(command))
    {
        PostLog(logName + L": ошибка удаления");
        return false;
    }

    PostLog(logName + L": команда выполнена");
    return true;
}

static bool UninstallSystemInformerIfPresent()
{
    return UninstallProgramIfPresent(
        L"SystemInformer",
        { L"system informer", L"systeminformer" },
        false);
}

static bool UninstallRecuvaIfPresent()
{
    return UninstallProgramIfPresent(
        L"Recuva",
        { L"recuva", L"piriform recuva", L"ccleaner recuva" },
        true);
}

static bool IsPathInsideFolder(const std::wstring& path, const std::wstring& folder)
{
    std::wstring p = ToLower(path);
    std::wstring f = ToLower(folder);

    while (!f.empty() && (f.back() == L'\\' || f.back() == L'/'))
        f.pop_back();

    if (p.size() <= f.size())
        return false;

    if (p.compare(0, f.size(), f) != 0)
        return false;

    return p[f.size()] == L'\\' || p[f.size()] == L'/';
}

static void CloseProcessesFromFolder(const std::wstring& folder)
{
    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

    if (snapshot == INVALID_HANDLE_VALUE)
        return;

    DWORD currentPid = GetCurrentProcessId();
    PROCESSENTRY32W pe{};
    pe.dwSize = sizeof(pe);

    if (Process32FirstW(snapshot, &pe))
    {
        do
        {
            if (pe.th32ProcessID == currentPid)
                continue;

            HANDLE process = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION | PROCESS_TERMINATE, FALSE, pe.th32ProcessID);

            if (!process)
                continue;

            wchar_t imagePath[MAX_PATH * 4]{};
            DWORD size = ARRAYSIZE(imagePath);

            if (QueryFullProcessImageNameW(process, 0, imagePath, &size))
            {
                if (IsPathInsideFolder(imagePath, folder))
                {
                    PostLog(L"Закрывается: " + std::wstring(pe.szExeFile));
                    TerminateProcess(process, 0);
                    WaitForSingleObject(process, 3000);
                }
            }

            CloseHandle(process);
        }
        while (Process32NextW(snapshot, &pe));
    }

    CloseHandle(snapshot);
}

static void DeleteHolyCheckWorker()
{
    RequestCancelDownloads();

    AbortActiveHttpHandles();

    for (int i = 0; i < 35 && (g_working.load() || g_activeSingleDownloads.load() > 0); ++i)
        Sleep(100);

    std::wstring folder = GetHolyCheckFolderPathOnly();

    PostLog(L"Закрываю программы");
    CloseProcessesFromFolder(folder);

    UninstallSystemInformerIfPresent();
    UninstallRecuvaIfPresent();

    CloseProcessesFromFolder(folder);

    if (DeleteDirectoryTree(folder))
        PostLog(L"Папка удалена полностью");
    else
        PostLog(L"Ошибка удаления папки");

    ClearActiveDownloads();
    g_deleteInProgress = false;
    PostProgress(0);

    if (g_hMain && !g_destroying.load())
        PostMessageW(g_hMain, WM_APP_DONE, 0, 0);
}

static void DeleteHolyCheckFolder()
{
    if (g_deleteInProgress.exchange(true))
        return;

    g_cancelRequested = true;
    SetDownloadControlsEnabled(false);
    g_progressPercent = 0;
    ClearDownloadLog();
    PostLog(L"Удаление holycheck");
    InvalidateBottomPanel();

    std::thread(DeleteHolyCheckWorker).detach();
}

static void InitParticles(int width, int height)
{
    g_particles.clear();

    int safeW = std::max(width, 1);
    int safeH = std::max(height, 1);
    int safeParticleH = std::max(safeH - 70, 1);

    if (safeW < 100 || safeH < 100)
        return;

    srand(static_cast<unsigned int>(time(nullptr)));

    for (int i = 0; i < 90; ++i)
    {
        Particle p{};

        p.x = 2.0f + static_cast<float>(rand() % std::max(safeW - 4, 1));
        p.y = 2.0f + static_cast<float>(rand() % std::max(safeParticleH - 4, 1));

        float speedX = ((rand() % 200) - 100) / 190.0f;
        float speedY = ((rand() % 200) - 100) / 190.0f;

        if (speedX > -0.06f && speedX < 0.06f)
            speedX = 0.16f;

        if (speedY > -0.06f && speedY < 0.06f)
            speedY = -0.16f;

        p.vx = speedX;
        p.vy = speedY;
        p.size = 1.2f + static_cast<float>(rand() % 3);
        p.alpha = static_cast<BYTE>(42 + rand() % 70);

        g_particles.push_back(p);
    }
}

static void UpdateParticles(HWND hwnd)
{
    if (g_particlesPaused || IsIconic(hwnd) || !IsWindowVisible(hwnd))
        return;

    RECT rc{};
    GetClientRect(hwnd, &rc);

    int width = static_cast<int>(rc.right - rc.left);
    int height = static_cast<int>(rc.bottom - rc.top);

    if (width < 100 || height < 100)
        return;

    if (g_particles.empty())
        InitParticles(width, height);

    const float minX = 2.0f;
    const float minY = 2.0f;
    const float maxX = static_cast<float>(std::max(width - 6, 4));
    const float maxY = static_cast<float>(std::max(height - 70, 4));

    bool needReinit = false;

    for (const auto& p : g_particles)
    {
        if (p.x < -20.0f || p.x > maxX + 20.0f || p.y < -20.0f || p.y > maxY + 20.0f)
        {
            needReinit = true;
            break;
        }
    }

    if (needReinit)
    {
        InitParticles(width, height);
    }

    for (auto& p : g_particles)
    {
        p.x += p.vx;
        p.y += p.vy;

        if (p.x < minX) p.x = maxX;
        if (p.x > maxX) p.x = minX;
        if (p.y < minY) p.y = maxY;
        if (p.y > maxY) p.y = minY;
    }

    RedrawWindow(
        hwnd,
        nullptr,
        nullptr,
        RDW_INVALIDATE | RDW_NOERASE | RDW_NOCHILDREN
    );
}

static void DrawParticles(Graphics& g, const RECT& rc)
{
    int width = static_cast<int>(rc.right - rc.left);
    int height = static_cast<int>(rc.bottom - rc.top);

    if (g_particlesPaused || width < 100 || height < 100 || g_particles.empty())
        return;

    int clipHeight = std::max(height - 58, 1);

    GraphicsState state = g.Save();
    g.SetClip(Rect(0, 0, static_cast<INT>(width), static_cast<INT>(clipHeight)), CombineModeReplace);
    g.SetSmoothingMode(SmoothingModeAntiAlias);

    Pen linePen(Color(14, 120, 130, 160), 1.0f);

    for (size_t i = 0; i < g_particles.size(); ++i)
    {
        for (size_t j = i + 1; j < g_particles.size(); ++j)
        {
            float dx = g_particles[i].x - g_particles[j].x;
            float dy = g_particles[i].y - g_particles[j].y;
            float dist2 = dx * dx + dy * dy;

            if (dist2 < 3600.0f)
            {
                g.DrawLine(
                    &linePen,
                    g_particles[i].x,
                    g_particles[i].y,
                    g_particles[j].x,
                    g_particles[j].y
                );
            }
        }
    }

    for (const auto& p : g_particles)
    {
        if (p.x < 0.0f || p.y < 0.0f || p.x > width || p.y > clipHeight)
            continue;

        SolidBrush brush(Color(p.alpha, 120, 160, 255));
        g.FillEllipse(&brush, p.x, p.y, p.size, p.size);
    }

    g.Restore(state);
}

static void DrawAmbientBackground(Graphics& g, const RECT& rc)
{
    const float w = static_cast<float>(rc.right - rc.left);

    g.SetSmoothingMode(SmoothingModeAntiAlias);

    // Старый тёмный фон без сетки, но с мягким бело-синим светом.
    GraphicsPath blueGlow;
    blueGlow.AddEllipse(RectF(18.0f, 88.0f, 430.0f, 430.0f));
    PathGradientBrush blueBrush(&blueGlow);
    blueBrush.SetCenterColor(Color(44, 80, 135, 255));
    Color blueSurround[] = { Color(0, 80, 135, 255) };
    INT blueCount = 1;
    blueBrush.SetSurroundColors(blueSurround, &blueCount);
    g.FillPath(&blueBrush, &blueGlow);

    GraphicsPath whiteGlow;
    whiteGlow.AddEllipse(RectF(w - 360.0f, -160.0f, 420.0f, 420.0f));
    PathGradientBrush whiteBrush(&whiteGlow);
    whiteBrush.SetCenterColor(Color(24, 245, 248, 255));
    Color whiteSurround[] = { Color(0, 245, 248, 255) };
    INT whiteCount = 1;
    whiteBrush.SetSurroundColors(whiteSurround, &whiteCount);
    g.FillPath(&whiteBrush, &whiteGlow);
}

static void DrawLogo(Graphics& g)
{
    g.SetSmoothingMode(SmoothingModeAntiAlias);
    g.SetPixelOffsetMode(PixelOffsetModeHighQuality);

    SolidBrush blue(Color(245, 74, 134, 255));
    SolidBrush cutout(Color(255, 18, 18, 19));

    // Простая лупа: без внешней прозрачной обводки, но с нормальным прорезом.
    g.FillEllipse(&blue, 106.0f, 78.0f, 207.0f, 207.0f);
    g.FillEllipse(&cutout, 139.0f, 111.0f, 141.0f, 141.0f);

    // Ручка чуть уже, без лишних линий.
    PointF handle[] =
    {
        PointF(268.0f, 235.0f),
        PointF(333.0f, 300.0f),
        PointF(358.0f, 330.0f),
        PointF(349.0f, 344.0f),
        PointF(332.0f, 354.0f),
        PointF(303.0f, 332.0f),
        PointF(238.0f, 271.0f)
    };

    g.FillPolygon(&blue, handle, 7);
}

static void DrawTitle(Graphics& g, RECT rc)
{
    g.SetTextRenderingHint(TextRenderingHintAntiAliasGridFit);

    FontFamily family(L"Segoe UI");
    Font titleFont(&family, 48.0f, FontStyleBold, UnitPixel);
    SolidBrush glowBrush(Color(95, 115, 165, 255));
    SolidBrush brush(Color(255, 246, 248, 255));

    StringFormat fmt;
    fmt.SetAlignment(StringAlignmentCenter);
    fmt.SetLineAlignment(StringAlignmentCenter);

    RectF glowLayout(0.0f, 18.0f, static_cast<REAL>(rc.right), 62.0f);
    RectF layout(0.0f, 15.0f, static_cast<REAL>(rc.right), 62.0f);

    g.DrawString(L"HolyCheck", -1, &titleFont, glowLayout, &fmt, &glowBrush);
    g.DrawString(L"HolyCheck", -1, &titleFont, layout, &fmt, &brush);
}

static void DrawSectionSeparator(Graphics& g, RECT rc)
{
    g.SetSmoothingMode(SmoothingModeAntiAlias);

    Pen linePen(Color(120, 58, 58, 66), 1.0f);

    float right = static_cast<float>(rc.right - 24);

    if (right < 520.0f)
        right = 520.0f;

    g.DrawLine(&linePen, 520.0f, 166.0f, right, 166.0f);
}

static void DrawProgressBar(Graphics& g, RECT rc)
{
    const float marginX = 22.0f;
    const float barHeight = 16.0f;

    RECT logRectNative = GetDownloadLogRect(rc);
    const float y = static_cast<float>(logRectNative.top) +
        (static_cast<float>(LOG_PANEL_H) - barHeight) / 2.0f;

    const float logGap = 18.0f;
    float x = static_cast<float>(LOG_PANEL_X + LOG_PANEL_W) + logGap;
    const float cancelReserve = static_cast<float>(CANCEL_BUTTON_W) + 18.0f;
    float width = static_cast<float>(rc.right) - x - marginX - cancelReserve;

    if (width < 220.0f)
    {
        x = marginX;
        width = static_cast<float>(rc.right) - marginX * 2.0f - cancelReserve;
    }

    if (width <= 10.0f)
        return;

    g.SetSmoothingMode(SmoothingModeAntiAlias);

    RectF bgRect(x, y, width, barHeight);

    GraphicsPath bgPath;
    bgPath.AddArc(bgRect.X, bgRect.Y, 12.0f, 12.0f, 180.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 12.0f, bgRect.Y, 12.0f, 12.0f, 270.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 12.0f, bgRect.GetBottom() - 12.0f, 12.0f, 12.0f, 0.0f, 90.0f);
    bgPath.AddArc(bgRect.X, bgRect.GetBottom() - 12.0f, 12.0f, 12.0f, 90.0f, 90.0f);
    bgPath.CloseFigure();

    SolidBrush bgBrush(Color(255, 42, 42, 46));
    g.FillPath(&bgBrush, &bgPath);

    float fillWidth = (width * static_cast<float>(g_progressPercent)) / 100.0f;

    if (fillWidth > 0.5f)
    {
        RectF fillRect(x, y, fillWidth, barHeight);

        GraphicsPath fillPath;

        if (fillWidth < 14.0f)
        {
            fillPath.AddRectangle(fillRect);
        }
        else
        {
            fillPath.AddArc(fillRect.X, fillRect.Y, 12.0f, 12.0f, 180.0f, 90.0f);
            fillPath.AddArc(fillRect.GetRight() - 12.0f, fillRect.Y, 12.0f, 12.0f, 270.0f, 90.0f);
            fillPath.AddArc(fillRect.GetRight() - 12.0f, fillRect.GetBottom() - 12.0f, 12.0f, 12.0f, 0.0f, 90.0f);
            fillPath.AddArc(fillRect.X, fillRect.GetBottom() - 12.0f, 12.0f, 12.0f, 90.0f, 90.0f);
            fillPath.CloseFigure();
        }

        SolidBrush fillBrush(Color(255, 128, 128, 140));
        g.FillPath(&fillBrush, &fillPath);
    }

    FontFamily family(L"Segoe UI");
    Font font(&family, 11.0f, FontStyleBold, UnitPixel);
    SolidBrush textBrush(Color(255, 235, 235, 240));

    std::wstring percentText = std::to_wstring(g_progressPercent) + L"%";

    StringFormat fmt;
    fmt.SetAlignment(StringAlignmentCenter);
    fmt.SetLineAlignment(StringAlignmentCenter);

    RectF textRect(x, y - 1.0f, width, barHeight + 2.0f);
    g.DrawString(percentText.c_str(), -1, &font, textRect, &fmt, &textBrush);
}

static void DrawDownloadLog(Graphics& g, RECT rc)
{
    RECT panelRect = GetDownloadLogRect(rc);

    if (rc.right < panelRect.right + 24)
        return;

    std::vector<std::wstring> lines;
    int scrollOffset = 0;

    {
        std::lock_guard<std::mutex> lock(g_logMutex);
        lines = g_logLines;
        ClampLogScrollOffsetLocked();
        scrollOffset = g_logScrollOffset;
    }

    std::wstring activeText = GetActiveDownloadsText();

    if (lines.empty() && activeText.empty())
        return;

    g.SetSmoothingMode(SmoothingModeAntiAlias);
    g.SetTextRenderingHint(TextRenderingHintClearTypeGridFit);

    RectF bgRect(
        static_cast<REAL>(panelRect.left),
        static_cast<REAL>(panelRect.top),
        static_cast<REAL>(panelRect.right - panelRect.left),
        static_cast<REAL>(panelRect.bottom - panelRect.top)
    );

    GraphicsPath bgPath;
    bgPath.AddArc(bgRect.X, bgRect.Y, 14.0f, 14.0f, 180.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 14.0f, bgRect.Y, 14.0f, 14.0f, 270.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 14.0f, bgRect.GetBottom() - 14.0f, 14.0f, 14.0f, 0.0f, 90.0f);
    bgPath.AddArc(bgRect.X, bgRect.GetBottom() - 14.0f, 14.0f, 14.0f, 90.0f, 90.0f);
    bgPath.CloseFigure();

    SolidBrush bgBrush(Color(125, 28, 30, 36));
    Pen borderPen(Color(132, 122, 128, 142), 1.0f);

    g.FillPath(&bgBrush, &bgPath);
    g.DrawPath(&borderPen, &bgPath);

    FontFamily family(L"Segoe UI");
    Font titleFont(&family, 12.0f, FontStyleBold, UnitPixel);
    Font lineFont(&family, 11.0f, FontStyleRegular, UnitPixel);
    SolidBrush titleBrush(Color(245, 236, 238, 245));
    SolidBrush textBrush(Color(255, 235, 235, 240));
    SolidBrush hintBrush(Color(210, 160, 160, 168));

    StringFormat fmt;
    fmt.SetTrimming(StringTrimmingEllipsisCharacter);
    fmt.SetFormatFlags(StringFormatFlagsNoWrap);

    const float x = static_cast<float>(panelRect.left);
    const float y = static_cast<float>(panelRect.top);
    const float panelW = static_cast<float>(panelRect.right - panelRect.left);
    const float panelH = static_cast<float>(panelRect.bottom - panelRect.top);

    RectF titleRect(x + 10.0f, y + 7.0f, panelW - 20.0f, 16.0f);
    g.DrawString(L"Лог загрузки", -1, &titleFont, titleRect, &fmt, &titleBrush);

    float firstLineY = y + 28.0f;

    if (!activeText.empty())
    {
        SolidBrush activeBrush(Color(255, 245, 245, 248));
        RectF activeRect(x + 10.0f, y + 25.0f, panelW - 20.0f, 16.0f);
        g.DrawString(activeText.c_str(), -1, &lineFont, activeRect, &fmt, &activeBrush);
        firstLineY = y + 45.0f;
    }

    if (lines.size() > LOG_VISIBLE_LINES)
    {
        StringFormat rightFmt;
        rightFmt.SetAlignment(StringAlignmentFar);
        rightFmt.SetTrimming(StringTrimmingEllipsisCharacter);
        rightFmt.SetFormatFlags(StringFormatFlagsNoWrap);

        RectF hintRect(x + 10.0f, y + 7.0f, panelW - 20.0f, 16.0f);
        g.DrawString(L"колесо мыши", -1, &lineFont, hintRect, &rightFmt, &hintBrush);
    }

    size_t visibleLines = std::min(LOG_VISIBLE_LINES, lines.size());
    size_t endIndex = lines.size();

    if (scrollOffset > 0)
    {
        size_t offset = static_cast<size_t>(scrollOffset);
        endIndex = offset < lines.size() ? lines.size() - offset : visibleLines;
    }

    if (endIndex > lines.size())
        endIndex = lines.size();

    size_t startIndex = endIndex > visibleLines ? endIndex - visibleLines : 0;

    float lineY = firstLineY;
    const float lineH = 13.0f;

    for (size_t i = startIndex; i < endIndex; ++i)
    {
        RectF textRect(x + 10.0f, lineY, panelW - 20.0f, lineH + 2.0f);
        g.DrawString(lines[i].c_str(), -1, &lineFont, textRect, &fmt, &textBrush);
        lineY += lineH;

        if (lineY > y + panelH - 8.0f)
            break;
    }
}

static std::wstring PreviewText(const wchar_t* text, size_t maxLen)
{
    std::wstring s = text ? text : L"";

    for (wchar_t& ch : s)
    {
        if (ch == L'\r' || ch == L'\n' || ch == L'\t')
            ch = L' ';
    }

    if (s.size() > maxLen)
        s = s.substr(0, maxLen - 3) + L"...";

    return s;
}

static void DrawHelperTextPanel(Graphics& g, RECT rc)
{
    RECT panelRect = GetHelperTextRect(rc);

    g.SetSmoothingMode(SmoothingModeAntiAlias);
    g.SetTextRenderingHint(TextRenderingHintClearTypeGridFit);

    RectF bgRect(
        static_cast<REAL>(panelRect.left),
        static_cast<REAL>(panelRect.top),
        static_cast<REAL>(panelRect.right - panelRect.left),
        static_cast<REAL>(panelRect.bottom - panelRect.top)
    );

    GraphicsPath bgPath;
    bgPath.AddArc(bgRect.X, bgRect.Y, 16.0f, 16.0f, 180.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 16.0f, bgRect.Y, 16.0f, 16.0f, 270.0f, 90.0f);
    bgPath.AddArc(bgRect.GetRight() - 16.0f, bgRect.GetBottom() - 16.0f, 16.0f, 16.0f, 0.0f, 90.0f);
    bgPath.AddArc(bgRect.X, bgRect.GetBottom() - 16.0f, 16.0f, 16.0f, 90.0f, 90.0f);
    bgPath.CloseFigure();

    SolidBrush bgBrush(Color(92, 30, 34, 45));
    Pen borderPen(Color(92, 120, 124, 136), 1.0f);
    g.FillPath(&bgBrush, &bgPath);
    g.DrawPath(&borderPen, &bgPath);

    FontFamily family(L"Segoe UI");
    Font titleFont(&family, 12.0f, FontStyleBold, UnitPixel);
    Font rowFont(&family, 10.0f, FontStyleRegular, UnitPixel);
    SolidBrush titleBrush(Color(245, 238, 240, 246));
    SolidBrush textBrush(Color(228, 210, 214, 224));
    SolidBrush labelBrush(Color(235, 174, 188, 218));

    StringFormat fmt;
    fmt.SetTrimming(StringTrimmingEllipsisCharacter);
    fmt.SetFormatFlags(StringFormatFlagsNoWrap);

    const float x = static_cast<float>(panelRect.left);
    const float y = static_cast<float>(panelRect.top);
    const float w = static_cast<float>(panelRect.right - panelRect.left);

    g.DrawString(L"Копирование", -1, &titleFont, RectF(x + 10.0f, y + 8.0f, w - 20.0f, 16.0f), &fmt, &titleBrush);

    struct Row { const wchar_t* label; const wchar_t* value; } rows[] =
    {
        { L"Читы", EVERYTHING_QUERY_1 },
        { L"Конфиги", EVERYTHING_QUERY_2 },
        { L"Check", L"Проверка системы, служб, журналов и списка твинков" },
        { L"Сайты", POWERSHELL_SITES_TXT }
    };

    float rowY = y + 30.0f;
    for (const auto& row : rows)
    {
        g.DrawString(row.label, -1, &rowFont, RectF(x + 10.0f, rowY, 70.0f, 16.0f), &fmt, &labelBrush);
        std::wstring preview = PreviewText(row.value, 54);
        g.DrawString(preview.c_str(), -1, &rowFont, RectF(x + 82.0f, rowY, w - 178.0f, 16.0f), &fmt, &textBrush);
        rowY += 27.0f;
    }
}

static void DrawScene(HWND hwnd, HDC targetHdc)
{
    RECT rc{};
    GetClientRect(hwnd, &rc);

    int w = rc.right - rc.left;
    int h = rc.bottom - rc.top;

    if (w <= 0 || h <= 0)
        return;

    HDC memDC = CreateCompatibleDC(targetHdc);
    HBITMAP memBmp = CreateCompatibleBitmap(targetHdc, w, h);
    HGDIOBJ oldBmp = SelectObject(memDC, memBmp);

    HBRUSH bgBrush = CreateSolidBrush(BG);
    FillRect(memDC, &rc, bgBrush);
    DeleteObject(bgBrush);

    Graphics g(memDC);
    g.SetSmoothingMode(SmoothingModeAntiAlias);
    g.SetPixelOffsetMode(PixelOffsetModeHighQuality);
    g.SetTextRenderingHint(TextRenderingHintClearTypeGridFit);

    DrawAmbientBackground(g, rc);
    DrawParticles(g, rc);
    DrawLogo(g);
    DrawSectionSeparator(g, rc);
    DrawProgressBar(g, rc);
    DrawDownloadLog(g, rc);
    DrawHelperTextPanel(g, rc);

    BitBlt(targetHdc, 0, 0, w, h, memDC, 0, 0, SRCCOPY);

    SelectObject(memDC, oldBmp);
    DeleteObject(memBmp);
    DeleteDC(memDC);
}

static void MoveChildWindow(HWND child, int x, int y, int w, int h, BOOL repaint)
{
    if (!child)
        return;

    RECT current{};
    GetWindowRect(child, &current);

    HWND parent = GetParent(child);
    POINT pts[2] =
    {
        { current.left, current.top },
        { current.right, current.bottom }
    };

    MapWindowPoints(nullptr, parent, pts, 2);

    if (pts[0].x == x && pts[0].y == y &&
        pts[1].x - pts[0].x == w && pts[1].y - pts[0].y == h)
    {
        return;
    }

    MoveWindow(child, x, y, w, h, repaint);
}

static void SetChildVisible(HWND child, bool visible)
{
    if (!child)
        return;

    bool current = IsWindowVisible(child) != FALSE;

    if (current == visible)
        return;

    ShowWindow(child, visible ? SW_SHOWNA : SW_HIDE);
}

static void LayoutControls(HWND hwnd)
{
    RECT rc{};
    GetClientRect(hwnd, &rc);

    int width = rc.right;

    MoveChildWindow(g_hDeleteHolyCheck, 28, 22, 118, 34, TRUE);
    ShowWindow(g_hDeleteHolyCheck, SW_SHOWNA);

    const int topButtonW = 126;
    const int topButtonH = 34;
    const int topGap = 9;
    const int topY = 16;
    int powerX = width - 28 - topButtonW;
    int servicesX = powerX - topGap - topButtonW;

    MoveChildWindow(g_hEnableServices, servicesX, topY, topButtonW, topButtonH, TRUE);
    MoveChildWindow(g_hPowerShell, powerX, topY, topButtonW, topButtonH, TRUE);
    ShowWindow(g_hEnableServices, SW_SHOWNA);
    ShowWindow(g_hPowerShell, SW_SHOWNA);

    RECT logRect = GetDownloadLogRect(rc);
    int cancelX = width - 28 - CANCEL_BUTTON_W;
    int cancelY = logRect.top + (LOG_PANEL_H - CANCEL_BUTTON_H) / 2;
    MoveChildWindow(g_hCancelDownload, cancelX, cancelY, CANCEL_BUTTON_W, CANCEL_BUTTON_H, TRUE);
    ShowWindow(g_hCancelDownload, SW_SHOWNA);

    RECT helperRect = GetHelperTextRect(rc);
    int copyX = helperRect.right - COPY_BUTTON_W - 14;
    int copyY = helperRect.top + 33;
    MoveChildWindow(g_hCopyEverything1, copyX, copyY, COPY_BUTTON_W, COPY_BUTTON_H, TRUE);
    MoveChildWindow(g_hCopyEverything2, copyX, copyY + 28, COPY_BUTTON_W, COPY_BUTTON_H, TRUE);
    MoveChildWindow(g_hCopyPowerShell, copyX, copyY + 56, COPY_BUTTON_W, COPY_BUTTON_H, TRUE);
    MoveChildWindow(g_hCopySites, copyX, copyY + 84, COPY_BUTTON_W, COPY_BUTTON_H, TRUE);
    ShowWindow(g_hCopyEverything1, SW_SHOWNA);
    ShowWindow(g_hCopyEverything2, SW_SHOWNA);
    ShowWindow(g_hCopyPowerShell, SW_SHOWNA);
    ShowWindow(g_hCopySites, SW_SHOWNA);

    int rightX = 520;
    int tabsY = 74;

    int tabW = 124;
    int tabH = 30;
    int gap = 7;

    for (int i = 0; i < static_cast<int>(g_tabButtons.size()); ++i)
    {
        MoveChildWindow(
            g_tabButtons[i],
            rightX + i * (tabW + gap),
            tabsY,
            tabW,
            tabH,
            TRUE
        );
        ShowWindow(g_tabButtons[i], SW_SHOWNA);
    }

    MoveChildWindow(g_hDownloadAll, rightX, tabsY + 44, 148, 34, TRUE);
    ShowWindow(g_hDownloadAll, SW_SHOWNA);

    int tileW = 138;
    int tileH = 40;
    int gapX = 12;
    int gapY = 12;
    int tileStartY = tabsY + 104;

    int availableW = std::max(width - rightX - 28, tileW);
    int columns = std::max(3, (availableW + gapX) / (tileW + gapX));
    if (columns > 5)
        columns = 5;

    // Сначала скрываем все плитки неактивных вкладок. Это важно: иначе при
    // быстрой смене вкладок Windows может оставить на фоне старый owner-draw
    // рисунок кнопки, и кажется, что программы перемешались между вкладками.
    for (int i = 0; i < static_cast<int>(g_items.size()); ++i)
    {
        if (g_items[i].tab != g_currentTab)
            ShowWindow(g_tileButtons[i], SW_HIDE);
    }

    RECT clearRect{ rightX - 18, tabsY + 88, rc.right, rc.bottom - LOG_INVALIDATE_HEIGHT + 6 };
    RedrawWindow(
        hwnd,
        &clearRect,
        nullptr,
        RDW_INVALIDATE | RDW_ERASE | RDW_NOCHILDREN | RDW_UPDATENOW
    );

    int visible = 0;

    for (int i = 0; i < static_cast<int>(g_items.size()); ++i)
    {
        if (g_items[i].tab != g_currentTab)
            continue;

        HWND btn = g_tileButtons[i];
        int col = visible % columns;
        int row = visible / columns;

        MoveChildWindow(
            btn,
            rightX + col * (tileW + gapX),
            tileStartY + row * (tileH + gapY),
            tileW,
            tileH,
            TRUE
        );

        ShowWindow(btn, SW_SHOWNA);
        InvalidateRect(btn, nullptr, TRUE);
        ++visible;
    }

    for (HWND b : g_tabButtons)
        InvalidateRect(b, nullptr, TRUE);

    for (HWND b : g_tileButtons)
    {
        if (IsWindowVisible(b))
            InvalidateRect(b, nullptr, TRUE);
    }

    InvalidateRect(g_hDownloadAll, nullptr, TRUE);
    InvalidateRect(g_hCancelDownload, nullptr, TRUE);
    InvalidateRect(g_hDeleteHolyCheck, nullptr, TRUE);
    InvalidateRect(g_hEnableServices, nullptr, TRUE);
    InvalidateRect(g_hPowerShell, nullptr, TRUE);
    InvalidateRect(g_hCopyEverything1, nullptr, TRUE);
    InvalidateRect(g_hCopyEverything2, nullptr, TRUE);
    InvalidateRect(g_hCopyPowerShell, nullptr, TRUE);
    InvalidateRect(g_hCopySites, nullptr, TRUE);

    RedrawWindow(
        hwnd,
        nullptr,
        nullptr,
        RDW_INVALIDATE | RDW_ALLCHILDREN | RDW_ERASE | RDW_UPDATENOW
    );
}

static void PaintParentSceneForButton(HWND button, HDC hdc)
{
    UNREFERENCED_PARAMETER(button);

    // Раньше каждая owner-draw кнопка заново рисовала всю сцену окна через
    // DrawScene(). При старте и переключении вкладок это давало заметное
    // мигание и задержку появления кнопок. Для кнопок достаточно быстро
    // очистить локальный фон тем же базовым цветом.
    RECT rc{};
    GetClientRect(button, &rc);

    HBRUSH brush = CreateSolidBrush(BG);
    FillRect(hdc, &rc, brush);
    DeleteObject(brush);
}

static void DrawRoundButton(DRAWITEMSTRUCT* d)
{
    HDC hdc = d->hDC;
    RECT r = d->rcItem;

    int id = static_cast<int>(d->CtlID);

    bool pressed = (d->itemState & ODS_SELECTED) != 0;
    bool disabled = ((d->itemState & ODS_DISABLED) != 0) || !IsWindowEnabled(d->hwndItem);
    bool selected = false;

    if (id >= IDC_TAB_BASE && id < IDC_TAB_BASE + TAB_COUNT)
        selected = (id - IDC_TAB_BASE == g_currentTab);

    bool copyButton = id == IDC_COPY_EVERYTHING_1 ||
        id == IDC_COPY_EVERYTHING_2 ||
        id == IDC_COPY_POWERSHELL ||
        id == IDC_COPY_SITES;

    COLORREF fill = PANEL;
    COLORREF borderColor = BORDER;
    COLORREF textColor = TEXT;

    if (disabled)
    {
        fill = RGB(48, 48, 52);
        borderColor = RGB(62, 62, 68);
        textColor = RGB(145, 145, 152);
    }
    else if (selected)
    {
        fill = RGB(58, 58, 66);
        borderColor = RGB(135, 145, 160);
    }
    else if (pressed)
    {
        fill = PANEL_PRESSED;
        borderColor = RGB(105, 112, 126);
    }
    else
    {
        fill = PANEL;
        borderColor = BORDER;
    }

    if (!disabled && (id == IDC_POWERSHELL || id == IDC_ENABLE_SERVICES || id == IDC_DELETE_HOLYCHECK))
    {
        fill = pressed ? RGB(70, 72, 82) : RGB(52, 54, 62);
        borderColor = RGB(92, 98, 112);
    }

    if (!disabled && copyButton)
    {
        fill = pressed ? RGB(54, 56, 64) : RGB(42, 44, 50);
        borderColor = RGB(78, 84, 96);
    }

    PaintParentSceneForButton(d->hwndItem, hdc);

    HBRUSH br = CreateSolidBrush(fill);
    HPEN pen = CreatePen(PS_SOLID, 1, borderColor);
    HGDIOBJ oldBrush = SelectObject(hdc, br);
    HGDIOBJ oldPen = SelectObject(hdc, pen);

    int radius = copyButton ? 10 : 14;
    RoundRect(hdc, r.left, r.top, r.right, r.bottom, radius, radius);

    SelectObject(hdc, oldPen);
    SelectObject(hdc, oldBrush);
    DeleteObject(pen);
    DeleteObject(br);

    wchar_t text[256]{};
    GetWindowTextW(d->hwndItem, text, 256);

    SetBkMode(hdc, TRANSPARENT);
    SetTextColor(hdc, textColor);

    HFONT buttonFont = g_hFont;
    if (copyButton)
        buttonFont = g_hTinyFont ? g_hTinyFont : g_hSmallFont;

    HGDIOBJ oldFont = SelectObject(hdc, buttonFont);

    DrawTextW(
        hdc,
        text,
        -1,
        &r,
        DT_CENTER | DT_VCENTER | DT_SINGLELINE | DT_END_ELLIPSIS
    );

    SelectObject(hdc, oldFont);
}

static LRESULT CALLBACK ButtonNoEraseProc(
    HWND hwnd,
    UINT msg,
    WPARAM wParam,
    LPARAM lParam,
    UINT_PTR subclassId,
    DWORD_PTR refData)
{
    UNREFERENCED_PARAMETER(subclassId);
    UNREFERENCED_PARAMETER(refData);

    if (msg == WM_ERASEBKGND)
        return 1;

    if (msg == WM_NCDESTROY)
        RemoveWindowSubclass(hwnd, ButtonNoEraseProc, 1);

    return DefSubclassProc(hwnd, msg, wParam, lParam);
}

static void MakeButtonNoErase(HWND button)
{
    if (button)
    {
        // Не ставим WS_EX_TRANSPARENT: из-за него owner-draw кнопки иногда
        // появляются только после клика/перерисовки. Прозрачность делаем
        // вручную цветами в DrawRoundButton.
        LONG_PTR exStyle = GetWindowLongPtrW(button, GWL_EXSTYLE);
        SetWindowLongPtrW(button, GWL_EXSTYLE, exStyle & ~WS_EX_TRANSPARENT);
        SetWindowSubclass(button, ButtonNoEraseProc, 1, 0);
    }
}

static void OpenPowerShellMinecraft()
{
    std::wstring minecraftFolder = GetMinecraftFolder();
    DeleteFileW(CombinePath(minecraftFolder, L"powershell.txt").c_str());

    std::wstring escapedFolder = EscapePowerShellSingleQuoted(minecraftFolder);
    std::wstring command =
        L"-NoExit -Command \"Set-Location -LiteralPath '" + escapedFolder + L"'\"";

    ShellExecuteW(
        g_hMain,
        L"runas",
        L"powershell.exe",
        command.c_str(),
        minecraftFolder.c_str(),
        SW_SHOWNORMAL
    );
}

static void OpenCmdEnableServices()
{
    std::wstring command =
        L"/k sc config PcaSVC start= auto & sc start PcaSVC & "
        L"sc config DPS start= auto & sc start DPS & "
        L"sc config SysMain start= auto & sc start SysMain & "
        L"sc config EventLog start= auto & sc start EventLog & "
        L"sc config bam start= auto & sc start bam";

    ShellExecuteW(
        g_hMain,
        L"runas",
        L"cmd.exe",
        command.c_str(),
        nullptr,
        SW_SHOWNORMAL
    );
}

static LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
    switch (msg)
    {
    case WM_CREATE:
    {
        g_uiThreadId = GetCurrentThreadId();
        g_hMain = hwnd;
        g_destroying = false;

        BOOL dark = TRUE;
        DwmSetWindowAttribute(hwnd, DWMWA_USE_IMMERSIVE_DARK_MODE, &dark, sizeof(dark));

        LOGFONTW lf{};
        wcscpy_s(lf.lfFaceName, L"Segoe UI");

        lf.lfHeight = -15;
        lf.lfWeight = FW_NORMAL;
        g_hFont = CreateFontIndirectW(&lf);

        lf.lfHeight = -13;
        lf.lfWeight = FW_SEMIBOLD;
        g_hSmallFont = CreateFontIndirectW(&lf);

        lf.lfHeight = -11;
        lf.lfWeight = FW_SEMIBOLD;
        g_hTinyFont = CreateFontIndirectW(&lf);

        const wchar_t* tabs[] =
        {
            L"Мл.сотрудник",
            L"Сотрудник",
            L"Вед.сотрудник",
            L"Доп.программы"
        };

        for (int i = 0; i < TAB_COUNT; ++i)
        {
            HWND tab = CreateWindowExW(
                0,
                L"BUTTON",
                tabs[i],
                WS_CHILD | BS_OWNERDRAW,
                0, 0, 0, 0,
                hwnd,
                reinterpret_cast<HMENU>(IDC_TAB_BASE + i),
                g_hInst,
                nullptr
            );

            SendMessageW(tab, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
            MakeButtonNoErase(tab);
            g_tabButtons.push_back(tab);
        }

        g_hDeleteHolyCheck = CreateWindowExW(
            0,
            L"BUTTON",
            L"Удалить",
            WS_CHILD | BS_OWNERDRAW,
            0, 0, 0, 0,
            hwnd,
            reinterpret_cast<HMENU>(IDC_DELETE_HOLYCHECK),
            g_hInst,
            nullptr
        );

        SendMessageW(g_hDeleteHolyCheck, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
        MakeButtonNoErase(g_hDeleteHolyCheck);

        g_hCancelDownload = CreateWindowExW(
            0,
            L"BUTTON",
            L"Отмена",
            WS_CHILD | BS_OWNERDRAW,
            0, 0, 0, 0,
            hwnd,
            reinterpret_cast<HMENU>(IDC_CANCEL_DOWNLOAD),
            g_hInst,
            nullptr
        );

        SendMessageW(g_hCancelDownload, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
        MakeButtonNoErase(g_hCancelDownload);

        g_hDownloadAll = CreateWindowExW(
            0,
            L"BUTTON",
            L"Скачать все",
            WS_CHILD | BS_OWNERDRAW,
            0, 0, 0, 0,
            hwnd,
            reinterpret_cast<HMENU>(IDC_DOWNLOAD_ALL),
            g_hInst,
            nullptr
        );

        SendMessageW(g_hDownloadAll, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
        MakeButtonNoErase(g_hDownloadAll);

        g_hPowerShell = CreateWindowExW(
            0,
            L"BUTTON",
            L"PowerShell",
            WS_CHILD | BS_OWNERDRAW,
            0, 0, 0, 0,
            hwnd,
            reinterpret_cast<HMENU>(IDC_POWERSHELL),
            g_hInst,
            nullptr
        );

        SendMessageW(g_hPowerShell, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
        MakeButtonNoErase(g_hPowerShell);

        g_hEnableServices = CreateWindowExW(
            0,
            L"BUTTON",
            L"Вкл службы",
            WS_CHILD | BS_OWNERDRAW,
            0, 0, 0, 0,
            hwnd,
            reinterpret_cast<HMENU>(IDC_ENABLE_SERVICES),
            g_hInst,
            nullptr
        );

        SendMessageW(g_hEnableServices, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
        MakeButtonNoErase(g_hEnableServices);

        g_hCopyEverything1 = CreateWindowExW(0, L"BUTTON", L"Читы", WS_CHILD | BS_OWNERDRAW, 0, 0, 0, 0, hwnd, reinterpret_cast<HMENU>(IDC_COPY_EVERYTHING_1), g_hInst, nullptr);
        SendMessageW(g_hCopyEverything1, WM_SETFONT, reinterpret_cast<WPARAM>(g_hTinyFont), TRUE);
        MakeButtonNoErase(g_hCopyEverything1);

        g_hCopyEverything2 = CreateWindowExW(0, L"BUTTON", L"Конфиги", WS_CHILD | BS_OWNERDRAW, 0, 0, 0, 0, hwnd, reinterpret_cast<HMENU>(IDC_COPY_EVERYTHING_2), g_hInst, nullptr);
        SendMessageW(g_hCopyEverything2, WM_SETFONT, reinterpret_cast<WPARAM>(g_hTinyFont), TRUE);
        MakeButtonNoErase(g_hCopyEverything2);

        g_hCopyPowerShell = CreateWindowExW(0, L"BUTTON", L"Check", WS_CHILD | BS_OWNERDRAW, 0, 0, 0, 0, hwnd, reinterpret_cast<HMENU>(IDC_COPY_POWERSHELL), g_hInst, nullptr);
        SendMessageW(g_hCopyPowerShell, WM_SETFONT, reinterpret_cast<WPARAM>(g_hTinyFont), TRUE);
        MakeButtonNoErase(g_hCopyPowerShell);

        g_hCopySites = CreateWindowExW(0, L"BUTTON", L"Сайты", WS_CHILD | BS_OWNERDRAW, 0, 0, 0, 0, hwnd, reinterpret_cast<HMENU>(IDC_COPY_SITES), g_hInst, nullptr);
        SendMessageW(g_hCopySites, WM_SETFONT, reinterpret_cast<WPARAM>(g_hTinyFont), TRUE);
        MakeButtonNoErase(g_hCopySites);

        for (int i = 0; i < static_cast<int>(g_items.size()); ++i)
        {
            HWND tile = CreateWindowExW(
                0,
                L"BUTTON",
                g_items[i].name.c_str(),
                WS_CHILD | BS_OWNERDRAW | BS_NOTIFY,
                0, 0, 0, 0,
                hwnd,
                reinterpret_cast<HMENU>(IDC_TILE_BASE + i),
                g_hInst,
                nullptr
            );

            SendMessageW(tile, WM_SETFONT, reinterpret_cast<WPARAM>(g_hFont), TRUE);
            MakeButtonNoErase(tile);
            g_tileButtons.push_back(tile);
        }

        g_itemBusy.assign(g_items.size(), 0);

        RECT rc{};
        GetClientRect(hwnd, &rc);
        InitParticles(rc.right, rc.bottom);

        SetTimer(hwnd, TIMER_PARTICLES, 33, nullptr);
        LayoutControls(hwnd);

        return 0;
    }

    case WM_GETMINMAXINFO:
    {
        MINMAXINFO* info = reinterpret_cast<MINMAXINFO*>(lParam);
        info->ptMinTrackSize.x = 980;
        info->ptMinTrackSize.y = 570;
        return 0;
    }

    case WM_SIZE:
    {
        if (wParam == SIZE_MINIMIZED)
        {
            g_particlesPaused = true;
            KillTimer(hwnd, TIMER_PARTICLES);
            return 0;
        }

        bool wasPaused = g_particlesPaused;
        g_particlesPaused = false;

        LayoutControls(hwnd);

        if (wasPaused || wParam == SIZE_RESTORED || wParam == SIZE_MAXIMIZED)
        {
            RECT rc{};
            GetClientRect(hwnd, &rc);

            if (rc.right > 100 && rc.bottom > 100)
                InitParticles(static_cast<int>(rc.right), static_cast<int>(rc.bottom));

            SetTimer(hwnd, TIMER_PARTICLES, 33, nullptr);
        }

        RedrawWindow(hwnd, nullptr, nullptr, RDW_INVALIDATE | RDW_NOERASE | RDW_NOCHILDREN);
        return 0;
    }

    case WM_ENTERSIZEMOVE:
    {
        g_particlesPaused = true;
        KillTimer(hwnd, TIMER_PARTICLES);
        return 0;
    }

    case WM_EXITSIZEMOVE:
    {
        g_particlesPaused = false;
        SetTimer(hwnd, TIMER_PARTICLES, 33, nullptr);
        InvalidateRect(hwnd, nullptr, FALSE);
        return 0;
    }

    case WM_TIMER:
        if (wParam == TIMER_PARTICLES)
        {
            if (!g_particlesPaused && !IsIconic(hwnd))
                UpdateParticles(hwnd);

            return 0;
        }
        break;

    case WM_MOUSEWHEEL:
    {
        POINT pt{};
        pt.x = static_cast<SHORT>(LOWORD(lParam));
        pt.y = static_cast<SHORT>(HIWORD(lParam));
        ScreenToClient(hwnd, &pt);

        RECT rc{};
        GetClientRect(hwnd, &rc);
        RECT logRect = GetDownloadLogRect(rc);

        if (PointInRect(logRect, pt))
        {
            if (ScrollDownloadLog(GET_WHEEL_DELTA_WPARAM(wParam)))
            {
                RECT invalidateRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };
                InvalidateRect(hwnd, &invalidateRect, FALSE);
            }

            return 0;
        }

        break;
    }

    case WM_COMMAND:
    {
        int id = LOWORD(wParam);
        int code = HIWORD(wParam);

        if (id == IDC_DOWNLOAD_ALL)
        {
            DownloadCurrentTab();
            return 0;
        }

        if (id == IDC_POWERSHELL)
        {
            OpenPowerShellMinecraft();
            return 0;
        }

        if (id == IDC_ENABLE_SERVICES)
        {
            OpenCmdEnableServices();
            return 0;
        }

        if (id == IDC_DELETE_HOLYCHECK)
        {
            DeleteHolyCheckFolder();
            return 0;
        }

        if (id == IDC_CANCEL_DOWNLOAD)
        {
            RequestCancelDownloads();
            return 0;
        }

        if (id == IDC_COPY_EVERYTHING_1)
        {
            CopyHelperText(EVERYTHING_QUERY_1, L"Everything 1");
            return 0;
        }

        if (id == IDC_COPY_EVERYTHING_2)
        {
            CopyHelperText(EVERYTHING_QUERY_2, L"Everything 2");
            return 0;
        }

        if (id == IDC_COPY_POWERSHELL)
        {
            CopyHelperText(POWERSHELL_TXT, L"PowerShell");
            return 0;
        }

        if (id == IDC_COPY_SITES)
        {
            CopyHelperText(POWERSHELL_SITES_TXT, L"Сайты");
            return 0;
        }

        if (id >= IDC_TAB_BASE && id < IDC_TAB_BASE + TAB_COUNT)
        {
            g_currentTab = id - IDC_TAB_BASE;
            LayoutControls(hwnd);

            return 0;
        }

        if (id >= IDC_TILE_BASE && id < IDC_TILE_BASE + static_cast<int>(g_items.size()))
        {
            if (code == BN_CLICKED)
                OpenOrDownloadSingleById(id);

            return 0;
        }

        break;
    }

    case WM_DRAWITEM:
    {
        DRAWITEMSTRUCT* d = reinterpret_cast<DRAWITEMSTRUCT*>(lParam);

        if (d)
        {
            DrawRoundButton(d);
            return TRUE;
        }

        break;
    }

    case WM_ERASEBKGND:
        return 1;

    case WM_PAINT:
    {
        PAINTSTRUCT ps{};
        HDC hdc = BeginPaint(hwnd, &ps);
        DrawScene(hwnd, hdc);
        EndPaint(hwnd, &ps);
        return 0;
    }

    case WM_APP_REFRESH:
    {
        RECT rc{};
        GetClientRect(hwnd, &rc);
        RECT panelRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };
        InvalidateRect(hwnd, &panelRect, FALSE);
        return 0;
    }

    case WM_APP_PROGRESS:
    {
        g_progressPercent = static_cast<int>(wParam);

        RECT rc{};
        GetClientRect(hwnd, &rc);
        RECT progressRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };
        InvalidateRect(hwnd, &progressRect, FALSE);

        return 0;
    }

    case WM_APP_LOG:
    {
        std::wstring* line = reinterpret_cast<std::wstring*>(lParam);

        if (line)
        {
            PushLogLine(*line);
            delete line;
        }

        RECT rc{};
        GetClientRect(hwnd, &rc);
        RECT logRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };
        InvalidateRect(hwnd, &logRect, FALSE);

        return 0;
    }

    case WM_APP_ITEM_DONE:
    {
        int index = static_cast<int>(wParam);
        SetItemBusyIndex(index, false);

        if (!g_deleteInProgress.load())
            SetTileEnabledByIndex(index, true);

        InvalidateBottomPanel();
        return 0;
    }

    case WM_APP_DONE:
    {
        g_working = false;

        RECT rc{};
        GetClientRect(hwnd, &rc);
        RECT progressRect{ 0, rc.bottom - LOG_INVALIDATE_HEIGHT, rc.right, rc.bottom };

        if (g_deleteInProgress.load())
        {
            g_progressPercent = 0;
            InvalidateRect(hwnd, &progressRect, FALSE);
            return 0;
        }

        SetDownloadControlsEnabled(true);

        if (IsCancelRequested())
            g_progressPercent = 0;
        else
            g_progressPercent = 100;

        g_cancelRequested = false;

        InvalidateRect(hwnd, &progressRect, FALSE);
        return 0;
    }

    case WM_DESTROY:
    {
        g_destroying = true;
        g_cancelRequested = true;
        AbortActiveHttpHandles();
        KillTimer(hwnd, TIMER_PARTICLES);

        if (g_hFont)
        {
            DeleteObject(g_hFont);
            g_hFont = nullptr;
        }

        if (g_hSmallFont)
        {
            DeleteObject(g_hSmallFont);
            g_hSmallFont = nullptr;
        }

        if (g_hTinyFont)
        {
            DeleteObject(g_hTinyFont);
            g_hTinyFont = nullptr;
        }

        g_hMain = nullptr;
        PostQuitMessage(0);
        return 0;
    }
    }

    return DefWindowProcW(hwnd, msg, wParam, lParam);
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE, PWSTR, int nCmdShow)
{
    g_hInst = hInstance;

    CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED);

    GdiplusStartupInput gdiplusStartupInput;
    GdiplusStartup(&g_gdiplusToken, &gdiplusStartupInput, nullptr);

    INITCOMMONCONTROLSEX icc{};
    icc.dwSize = sizeof(icc);
    icc.dwICC = ICC_WIN95_CLASSES | ICC_PROGRESS_CLASS;
    InitCommonControlsEx(&icc);

    WNDCLASSEXW wc{};
    wc.cbSize = sizeof(wc);
    wc.style = CS_DBLCLKS;
    wc.lpfnWndProc = WndProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = L"HolyCheckDownloaderClass";
    wc.hCursor = LoadCursorW(nullptr, IDC_ARROW);
    wc.hIcon = LoadIconW(nullptr, IDI_APPLICATION);
    wc.hIconSm = LoadIconW(nullptr, IDI_APPLICATION);
    wc.hbrBackground = nullptr;

    if (!RegisterClassExW(&wc))
    {
        MessageBoxW(nullptr, L"Не удалось зарегистрировать класс окна.", L"Ошибка", MB_ICONERROR);

        if (g_gdiplusToken)
            GdiplusShutdown(g_gdiplusToken);

        CoUninitialize();
        return 1;
    }

    g_hMain = CreateWindowExW(
        0,
        wc.lpszClassName,
        L"holycheck <3",
        WS_OVERLAPPEDWINDOW | WS_CLIPCHILDREN | WS_CLIPSIBLINGS,
        CW_USEDEFAULT,
        CW_USEDEFAULT,
        1160,
        660,
        nullptr,
        nullptr,
        hInstance,
        nullptr
    );

    if (!g_hMain)
    {
        MessageBoxW(nullptr, L"Не удалось создать окно.", L"Ошибка", MB_ICONERROR);

        if (g_gdiplusToken)
            GdiplusShutdown(g_gdiplusToken);

        CoUninitialize();
        return 1;
    }

    ShowWindow(g_hMain, nCmdShow);
    UpdateWindow(g_hMain);

    MSG msg{};

    while (GetMessageW(&msg, nullptr, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessageW(&msg);
    }

    if (g_gdiplusToken)
        GdiplusShutdown(g_gdiplusToken);

    CoUninitialize();
    return 0;
}
