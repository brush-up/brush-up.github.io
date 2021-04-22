---
title:  "[C++]폴더내 모든 파일 검색 샘플 소스"
excerpt: "폴더내 모든 파일 검색 샘플 소스"

categories:
  - Cpp
tags:
  - [Cpp, Programming, MFC, Windows]

toc: true
toc_sticky: true
 
date: 2010-04-24
last_modified_at: 2010-04-24
---

# 샘플소스
```c++
CString strFindPath, strReadPath;
CString strFileName;
BOOL bContinue;
int file_count = 0;
strFindPath = ".\\EXPORT\\0725\\*.txt";//탐색할 폴더의 패스 지정
CFileFind filefind;
bContinue = filefind.FindFile(strFindPath);//첫번째 파일 가져오기
// 파일이 없으면..
if(!bContinue)
{
    filefind.Close();
    return;
}
// 파일 있을때 까지 루프
while(bContinue)
{
    file_count++;
    //다음 파일 이름 가져오기
    bContinue = filefind.FindNextFile();
    strReadPath = filefind.GetFileName();
    // 폴더일경우
    if(filefind.IsDirectory()){
        //. .. 인 경우.
        if(strReadPath != "." && strReadPath != ".."){
        }
    }else{//폴더가 아닐 경우
        strFileName = filefind.GetFileName();
        strReadPath = filefind.GetFilePath();
    }
}
filefind.Close();
```