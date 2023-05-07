//Page replacement algorithm LFU,MFU,LRU, Optimal, SecondChance

#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>

using namespace std;

int lfuPageReplacement(const vector<int>& referenceString, int numFrames) {
    vector<int> frames(numFrames, -1); // Initialize frames with -1 indicating empty frames
    unordered_map<int, int> pageToFrequency; // Mapping of page numbers to their frequency of use
    unordered_map<int, int> pageToFrame; // Mapping of page numbers to frame numbers
    int pageFaults = 0;

    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];

        // Check if the page is already in a frame
        if (pageToFrame.find(page) != pageToFrame.end()) {
            // Increment the frequency of use for the page
            pageToFrequency[page]++;
        }
        else {
            // Page fault occurred
            pageFaults++;

            if (frames.size() == numFrames) {
                // Find the page with the least frequency of use
                int pageToReplace = frames[0];
                int minFrequency = pageToFrequency[pageToReplace];

                for (int j = 1; j < numFrames; j++) {
                    int currentFramePage = frames[j];
                    int currentFrequency = pageToFrequency[currentFramePage];

                    if (currentFrequency < minFrequency) {
                        pageToReplace = currentFramePage;
                        minFrequency = currentFrequency;
                    }
                }

                // Remove the page with the least frequency of use from frames and pageToFrame
                auto it = find(frames.begin(), frames.end(), pageToReplace);
                frames.erase(it);
                pageToFrame.erase(pageToReplace);
                pageToFrequency.erase(pageToReplace);
            }
        }
        if (pageToFrame.find(page) == pageToFrame.end()) {
                // Add the current page to an empty frame or replace the least frequently used page
                frames.push_back(page);
                pageToFrame[page] = page;
                pageToFrequency[page] = 1;
            }
        }
        return pageFaults;
}

int mfuPageReplacement(const vector<int>& referenceString, int numFrames) {
    vector<int> frames(numFrames, -1); // Initialize frames with -1 indicating empty frames
    unordered_map<int, int> pageToFrequency; // Mapping of page numbers to their frequency of use
    unordered_map<int, int> pageToFrame; // Mapping of page numbers to frame numbers
    int pageFaults = 0;

    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];

        // Check if the page is already in a frame
        if (pageToFrame.find(page) != pageToFrame.end()) {
            // Increment the frequency of use for the page
            pageToFrequency[page]++;
        }
        else {
            // Page fault occurred
            pageFaults++;

            if (frames.size() == numFrames) {
                // Find the page with the highest frequency of use
                int pageToReplace = frames[0];
                int maxFrequency = pageToFrequency[pageToReplace];

                for (int j = 1; j < numFrames; j++) {
                    int currentFramePage = frames[j];
                    int currentFrequency = pageToFrequency[currentFramePage];

                    if (currentFrequency > maxFrequency) {
                        pageToReplace = currentFramePage;
                        maxFrequency = currentFrequency;
                    }
                }

                // Remove the page with the highest frequency of use from frames and pageToFrame
                auto it = find(frames.begin(), frames.end(), pageToReplace);
                frames.erase(it);
                pageToFrame.erase(pageToReplace);
                pageToFrequency.erase(pageToReplace);
            }
        }

        if (pageToFrame.find(page) == pageToFrame.end()) {
            // Add the current page to an empty frame or replace the page with the highest frequency of use
            frames.push_back(page);
            pageToFrame[page] = page;
            pageToFrequency[page] = 1;
        }
    }

    return pageFaults;
}



int lruPageReplacement(const vector<int>& referenceString, int numFrames) {
    vector<int> frames(numFrames, -1); // Initialize frames with -1 indicating empty frames
    unordered_map<int, int> pageToFrame; // Mapping of page numbers to frame numbers
    int pageFaults = 0;

    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];

        // Check if the page is already in a frame
        if (pageToFrame.find(page) != pageToFrame.end()) {
            // Update the frame's position in frames vector
            auto it = find(frames.begin(), frames.end(), pageToFrame[page]);
            frames.erase(it);
        }
        else {
            // Page fault occurred
            pageFaults++;

            if (frames.size() == numFrames) {
                // Remove the least recently used page from frames and pageToFrame
                pageToFrame.erase(frames.back());
                frames.pop_back();
            }
        }

        // Add the current page to the front of frames and update pageToFrame
        frames.insert(frames.begin(), page);
        pageToFrame[page] = page;
    }

    return pageFaults;
}



int optimalPageReplacement(const vector<int>& referenceString, int numFrames) {
    vector<int> frames(numFrames, -1); // Initialize frames with -1 indicating empty frames
    unordered_map<int, int> pageToNextIndex; // Mapping of page numbers to their next occurrence index
    int pageFaults = 0;

    // Initialize pageToNextIndex with -1 indicating the page doesn't occur again in the reference string
    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];
        if (pageToNextIndex.find(page) == pageToNextIndex.end()) {
            pageToNextIndex[page] = -1;
        }
    }

    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];

        // Check if the page is already in a frame
        if (find(frames.begin(), frames.end(), page) != frames.end()) {
            continue;
        }

        // Page fault occurred
        pageFaults++;

        if (frames.size() < numFrames) {
            // Add the current page to an empty frame
            frames[frames.size()] = page;
        }
        else {
            // Find the page in frames that will not be used for the longest duration
            int pageToReplace = -1;
            int maxNextIndex = -1;

            for (int j = 0; j < numFrames; j++) {
                int currentFramePage = frames[j];
                if (pageToNextIndex[currentFramePage] == -1) {
                    // This page will not occur again, so it can be replaced
                    pageToReplace = currentFramePage;
                    break;
                }
                else if (pageToNextIndex[currentFramePage] > maxNextIndex) {
                    maxNextIndex = pageToNextIndex[currentFramePage];
                    pageToReplace = currentFramePage;
                }
            }

            // Replace the page with the page to be replaced
            auto it = find(frames.begin(), frames.end(), pageToReplace);
            *it = page;
        }

        // Update the next occurrence index of the current page
        for (int j = i + 1; j < referenceString.size(); j++) {
            if (referenceString[j] == page) {
                pageToNextIndex[page] = j;
                break;
            }
        }
    }

    return pageFaults;
}

int secondChancePageReplacement(const vector<int>& referenceString, int numFrames) {
    vector<int> frames(numFrames, -1); // Initialize frames with -1 indicating empty frames
    vector<bool> secondChanceBit(numFrames, false); // Second chance bit for each frame
    unordered_map<int, int> pageToFrame; // Mapping of page numbers to frame numbers
    int pageFaults = 0;

    for (int i = 0; i < referenceString.size(); i++) {
        int page = referenceString[i];

        // Check if the page is already in a frame
        if (pageToFrame.find(page) != pageToFrame.end()) {
            // Set the second chance bit of the frame to true
            int frame = pageToFrame[page];
            secondChanceBit[frame] = true;
        }
        else {
            // Page fault occurred
            pageFaults++;

            if (frames.size() < numFrames) {
                // Add the current page to an empty frame
                int emptyFrame = find(frames.begin(), frames.end(), -1) - frames.begin();
                frames[emptyFrame] = page;
                pageToFrame[page] = emptyFrame;
                secondChanceBit[emptyFrame] = true;
            }
            else {
                // Find a page to evict using the second chance algorithm
                bool pageEvicted = false;
                int evictedFrame;

                while (!pageEvicted) {
                    int currentIndex = 0;
                    while (true) {
                        int frame = currentIndex % numFrames;
                        int currentPage = frames[frame];

                        if (!secondChanceBit[frame]) {
                            // Evict the current page
                            evictedFrame = frame;
                            pageToFrame.erase(currentPage);
                            pageToFrame[page] = frame;
                            secondChanceBit[frame] = true;
                            pageEvicted = true;
                            break;
                        }
                        else {
                            // Set the second chance bit to false and move to the next frame
                            secondChanceBit[frame] = false;
                        }

                        currentIndex++;
                    }
                }

                // Replace the evicted page with the current page
                frames[evictedFrame] = page;
            }
        }
    }

    return pageFaults;
}

int main() {
    int numFrames;
    cout << "Enter the number of free frames: ";
    cin >> numFrames;

    int numReferences;
    cout << "Enter the number of page references: ";
    cin >> numReferences;

    vector<int> referenceString(numReferences);
    cout << "Enter the reference string: ";
    for (int i = 0; i < numReferences; i++) {
        cin >> referenceString[i];
    }
    //function call korbo proyojonmoto
    int lruPageFaults = lruPageReplacement(referenceString, numFrames);
    int optimalPageFaults = optimalPageReplacement(referenceString, numFrames);
    int secondChancepageFaults = secondChancePageReplacement(referenceString, numFrames);
    
    
    //pageFaults print korbo
    cout << "LRU Page Faults: " << lruPageFaults << endl;
    cout << "Optimal Page Faults: " << optimalPageFaults << endl;

    return 0;
}

