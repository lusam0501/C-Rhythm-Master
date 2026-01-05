# C-Rhythm-Master
```cpp
{
#include <iostream>
#include <vector>
#include <ctime>
#include <conio.h>
#include <ctype.h> 
#include "rlutil.h"

using namespace std;

struct Note {
    int x, y;
    char key;
};

// 繪製血條
void drawHPBar(int hp) {
    int barWidth = 10;
    int filled = (hp > 0) ? (hp / 10) : 0;
    cout << "[";
    for (int i = 0; i < barWidth; i++) {
        if (i < filled) cout << "=";
        else cout << " ";
    }
    cout << "] " << hp << "%";
}

// 根據字母返回顏色代碼
void setKeyColor(char c) {
    if (c == 'w') rlutil::setColor(rlutil::RED);
    else if (c == 'a') rlutil::setColor(rlutil::YELLOW);
    else if (c == 's') rlutil::setColor(rlutil::GREEN);
    else if (c == 'd') rlutil::setColor(rlutil::LIGHTBLUE);
    else rlutil::setColor(rlutil::WHITE);
}

int main() {
    srand(time(NULL));
    rlutil::saveDefaultColor();
    rlutil::hidecursor();
    
    // --- 1. 難度選擇 ---
    rlutil::cls();
    rlutil::setColor(rlutil::CYAN); // 選單標題保留青藍色
    cout << "==============================" << endl;
    cout << "   RHYTHM MASTER - SELECT MODE" << endl;
    cout << "==============================" << endl;
    rlutil::setColor(rlutil::WHITE);
    cout << "  1. EASY   (Slow / Less Notes)" << endl;
    cout << "  2. NORMAL (Standard)" << endl;
    cout << "  3. HARD   (Fast / Dense Notes)" << endl;
    cout << "==============================" << endl;
    cout << "Press 1, 2, or 3 to start..." << endl;

    int choice = 0;
    while (choice < '1' || choice > '3') { choice = _getch(); }

    int msleepTime = 80, spawnFreq = 6;    
    string modeName = "";
    if (choice == '1') { msleepTime = 120; spawnFreq = 10; modeName = "EASY"; }
    else if (choice == '2') { msleepTime = 80; spawnFreq = 6; modeName = "NORMAL"; }
    else if (choice == '3') { msleepTime = 50; spawnFreq = 4; modeName = "HARD"; }

    // --- 2. 倒數計時 ---
    for (int i = 3; i > 0; i--) {
        rlutil::cls();
        rlutil::locate(35, 10);
        rlutil::setColor(rlutil::LIGHTMAGENTA);
        cout << "READY? " << i << "..." << endl;
        rlutil::msleep(1000);
    }

    // --- 3. 遊戲初始化 ---
    int score = 0, combo = 0, maxCombo = 0, hp = 100;
    vector<Note> notes;
    int loopCounter = 0;
    int lanes[4] = {20, 30, 40, 50};
    char laneKeys[4] = {'w', 'a', 's', 'd'};
    int targetLine = 18;
    int bottomLine = 20;

    // --- 4. 遊戲主迴圈 ---
    while (hp > 0) {
        if (_kbhit()) {
            int k = _getch();
            if (k == 27) break; 

            char pressedKey = tolower((char)k);
            for (int i = 0; i < (int)notes.size(); i++) {
                if (notes[i].y >= targetLine - 1 && notes[i].y <= targetLine + 1) {
                    if (pressedKey == notes[i].key) {
                        score += (10 + combo);
                        combo++;
                        if (combo > maxCombo) maxCombo = combo;
                        notes.erase(notes.begin() + i);
                        goto frame_update; 
                    }
                }
            }
        }

        if (loopCounter % spawnFreq == 0) {
            int laneIdx = rand() % 4;
            Note n = {lanes[laneIdx], 1, laneKeys[laneIdx]};
            notes.push_back(n);
        }

        frame_update:
        rlutil::cls();
        
        // --- 繪製資訊欄 (修正為白色) ---
        rlutil::setColor(rlutil::WHITE); 
        rlutil::locate(60, 4);  cout << "MODE:  " << modeName;
        rlutil::locate(60, 6);  cout << "SCORE: " << score;
        rlutil::locate(60, 8);  cout << "COMBO: " << combo;
        rlutil::locate(60, 10); cout << "MAX  : " << maxCombo;
        rlutil::locate(60, 12); cout << "HP:    "; drawHPBar(hp);
        rlutil::setColor(rlutil::GREY);
        rlutil::locate(60, 20); cout << "[ESC] TO QUIT";

        // --- 繪製彩色分隔線 ---
        rlutil::setColor(1 + (loopCounter % 15)); 
        rlutil::locate(1, targetLine);
        for(int i = 0; i < 58; i++) cout << "-";

        // --- 繪製軌道與字母標示 ---
        for (int i = 0; i < 4; i++) {
            rlutil::setColor(rlutil::WHITE);
            rlutil::locate(lanes[i], targetLine); cout << "===";
            setKeyColor(laneKeys[i]); 
            rlutil::locate(lanes[i], targetLine + 2); cout << "[" << (char)toupper(laneKeys[i]) << "]";
        }
        
        // --- 繪製下落方塊 ---
        for (int i = 0; i < (int)notes.size(); i++) {
            notes[i].y++;
            if (notes[i].y >= bottomLine) {
                hp -= 10;
                combo = 0;
                notes.erase(notes.begin() + i);
                i--;
                continue;
            }
            rlutil::locate(notes[i].x + 1, notes[i].y);
            setKeyColor(notes[i].key); 
            cout << notes[i].key;
        }

        loopCounter++;
        rlutil::msleep(msleepTime);
    }

    // --- 5. 結算畫面 ---
    rlutil::cls();
    rlutil::setColor(rlutil::CYAN);
    cout << "==================================" << endl;
    cout << "           GAME RESULT            " << endl;
    cout << "==================================" << endl;
    rlutil::setColor(rlutil::WHITE);
    cout << "  MODE       : " << modeName << endl;
    cout << "  FINAL SCORE: " << score << endl;
    cout << "  MAX COMBO  : " << maxCombo << endl;
    rlutil::setColor(rlutil::CYAN);
    cout << "==================================" << endl;
    rlutil::resetColor();
    cout << "Press any key to exit..." << endl;
    _getch();

    return 0;
}
}
```
