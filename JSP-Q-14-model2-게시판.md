# 1. 바꿔 말하면 이 실습 게시판에서 db에 업로드 된 내용은 남아있어도 다음 날 다시 켜면 다운로드가 안된다는 뜻인가?

네, **정확히 그렇게 됩니다.** 😟

데이터베이스(DB)에 파일 정보가 남아 있더라도, 실제 파일이 저장된 위치가 **임시 폴더**였기 때문에 다음 날 서버를 다시 켜면 다운로드가 안 됩니다.

---

## 📌 왜 다운로드가 안 되는가?

이 현상을 이해하는 데 중요한 두 가지 요소가 있습니다:

### 1. DB에는 기록이 남아있다 (파일의 '이름'만)

DB에는 다음과 같은 정보가 영구적으로 남아 있습니다.

* **`ofile`**: 원본 파일명 (예: `여행계획.pdf`)
* **`sfile`**: 서버 저장 파일명 (예: `20251124_XYZ123.pdf`)
* **`idx`**: 게시글 번호

이 정보 덕분에 게시글 목록에서는 파일명이 계속 보입니다.

### 2. 실제 파일은 사라진다 (파일의 '본체')

하지만 실제 파일(`.pdf` 본체)은 IDE가 서버 실행을 위해 만든 **임시 폴더** (`.../tmp0/wtpwebapps/MustHaveJSP/Uploads`)에 저장되었습니다.

* IDE를 끄거나 서버를 **재시작/클린**하는 과정에서 이 **`tmp0` 임시 폴더가 초기화**되거나 삭제됩니다.
* DB에 저장된 `sfile`을 가지고 다운로드를 시도하면, `DownController`는 **파일이 있어야 할 경로에서 파일 본체를 찾을 수 없기 때문에** (FileNotFound) 다운로드가 실패합니다.

---

## ✅ 실습 후 조치 사항

이 문제를 해결하고 실습 결과물을 보존하려면, 이전 답변에서 안내해 드린 대로 **반드시 서버 외부에 영구적인 업로드 경로를 설정**해야 합니다.

예: `C:\ProjectFiles\Uploads`

이 경로를 사용하도록 코드를 수정해야 다음 날에도 DB에 기록된 파일 정보를 이용해 실제 파일을 다운로드할 수 있습니다.

또한 모든 앱의 전체 기능을 이용하려면 [Gemini 앱 활동](https://myactivity.google.com/product/gemini)을 사용 설정하세요.

---

# 2. 예외 에러

제공해주신 `EditController.java`와 `FileUtil.java` 코드를 확인했습니다.

말씀하신 `StringIndexOutOfBoundsException`은 예상했던 대로 **`FileUtil.renameFile()` 메서드에 빈 문자열이 전달**되면서 발생하고 있습니다.

문제는 \*\*`EditController.java`\*\*의 파일 처리 로직과 \*\*`FileUtil.java`\*\*의 파일 이름 변경 로직 **모두**에 있습니다.

-----

## 🛠️ `StringIndexOutOfBoundsException` 해결 방안

아래 두 단계를 통해 문제를 해결할 수 있습니다.

### 1\. `FileUtil.java` 수정: 안전한 `renameFile` 메서드 (필수)

가장 근본적인 해결책은 `renameFile` 메서드 자체를 수정하여, **파일 이름에 마침표(`.`)가 없거나 빈 문자열일 때** 예외가 발생하지 않도록 방어하는 것입니다.

`FileUtil.java` 파일의 `renameFile` 메서드를 아래와 같이 수정해 주세요.

```java
// FileUtil.java - renameFile 메서드 수정
public static String renameFile(String sDirectory, String fileName) {
    // 💡 추가된 부분: 파일 이름이 null이거나 비어있거나, 마침표가 없을 경우
    int lastDot = fileName.lastIndexOf(".");
    
    // 파일명 변경이 불가능하거나 불필요한 경우 (즉, 마침표가 없거나 파일명이 비어있는 경우)
    if (fileName == null || fileName.isEmpty() || lastDot == -1) {
        // 기존 이름 그대로 반환 (상위 컨트롤러에서 처리하도록 유도)
        return fileName; 
    }
    
    // ext : 파일 확장자 저장 (lastDot은 이제 0 이상임이 보장됨)
    String ext = fileName.substring(lastDot);
    
    // now : 오늘 날짜 및 시간 문자열로 저장
    String now = new SimpleDateFormat("yyyyMMdd_HmsS").format(new Date());
    
    // ... (이하 로직은 동일)
    
    String newFileName = now + ext;
    // ... (파일 객체 생성 및 renameTo 로직)
    
    File oldFile = new File(sDirectory + File.separator + fileName);
    File newFile = new File(sDirectory + File.separator + newFileName);
    //원본 파일의 이름을 새로운파일의 이름으로 교체
    oldFile.renameTo(newFile);
    return newFileName;
}
```

-----

**파일을 새로 첨부하지 않고 수정할 때** `renameFile`이 실행되지 않거나, 실행되더라도 안전하게 빈 문자열을 처리하여 `StringIndexOutOfBoundsException`이 더 이상 발생하지 않을 것입니다.

# 업로드된 파일이 있는 글을 업로드가 아닌 글로 수정했을 경우에 db랑 업로드된 파일은 어떻게 되는거지? 이 코드에서

제공해주신 `EditController.java` 코드 기준으로, **업로드된 파일이 있는 글을 파일을 첨부하지 않은 글(업로드가 없는 글)로 수정했을 때** 데이터베이스(DB)와 서버에 업로드된 파일의 상태는 다음과 같이 처리됩니다.

-----

## 💾 파일 처리 및 DB 상태 분석

### 1\. 📂 서버에 업로드된 파일 (기존 파일)

**기존 파일은 삭제됩니다.**

`EditController.doPost` 메서드의 파일 처리 로직은 다음과 같습니다:

```java
// EditController.java (수정된 FileUtil.renameFile을 사용한다고 가정)

if (originalFileName != null && !originalFileName.isEmpty()) { 
    // originalFileName은 사용자가 새로 첨부한 파일의 이름입니다.
    // 파일을 새로 첨부하지 않았다면 이 조건문은 실행되지 않습니다.
    
} else { // 파일을 새로 첨부하지 않은 경우 (업로드가 없는 글로 수정)
    dto.setOfile(prevOfile);
    dto.setSfile(prevSfile);
    // 💡 FileUtil.deleteFile 로직이 실행되지 않습니다.
}
```

  * **문제**: 파일을 **새로 첨부하지 않고** 텍스트만 수정하거나 파일을 제거하고 싶을 경우, `originalFileName`이 빈 문자열(`""`)이 되므로 `else` 블록이 실행됩니다.
  * `else` 블록은 기존의 `prevOfile`과 `prevSfile` 값을 `dto`에 그대로 **유지**시킵니다.
  * 가장 중요한 점은 **기존 파일을 삭제하는 `FileUtil.deleteFile` 메서드가 호출되지 않는다는 것**입니다.

**결론**: 기존에 업로드되었던 파일(`prevSfile`)은 서버의 `Uploads` 폴더에 **그대로 남아 있게 됩니다.**

-----

### 2\. 📝 데이터베이스 (DB) 기록

**기존 파일 정보가 그대로 유지됩니다.**

`EditController`가 `else` 블록을 실행한 후, `dto` 객체는 다음과 같은 상태를 가집니다:

  * `dto.ofile` = `prevOfile` (이전 파일명)
  * `dto.sfile` = `prevSfile` (이전 서버 저장 파일명)

이후 `dao.updatePost(dto)`를 호출하면, **DB의 해당 게시글 레코드**에 있는 `ofile`과 `sfile` 필드는 **수정되기 전의 파일 이름**을 계속 가지고 있게 됩니다.

-----

## 💡 최종 결과 및 보안 문제

| 항목 | 결과 | 설명 |
| :--- | :--- | :--- |
| **서버 파일 (Uploads 폴더)** | **파일 남아있음** | 파일을 삭제하는 `deleteFile`이 호출되지 않았기 때문에, 불필요한 파일(좀비 파일)이 서버에 축적됩니다. |
| **DB 기록** | **파일 정보 유지** | `ofile`과 `sfile` 값이 그대로 유지되므로, 겉보기에는 파일이 첨부된 글처럼 보입니다. |

### 보안/관리 문제 (파일 삭제 로직 누락)

사용자가 수정 페이지에서 기존 파일을 **명시적으로 제거**하고 싶다면, 보통 수정 폼에 **"파일 삭제" 체크박스**를 추가하고, 체크박스가 활성화되었을 때 `deleteFile`을 호출해야 합니다.

현재 코드 구조에서는 파일 제거 옵션이 없기 때문에, 업로드가 없는 글이 되었음에도 DB에는 파일 정보가 남아있고, 서버에는 파일 본체가 남아 있어 **데이터 불일치**와 **서버 용량 낭비** 문제가 발생합니다.
