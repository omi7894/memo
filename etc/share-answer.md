
### 공람자 추가 - 이미 존재하는 열람자인지 확인
```
// 결재 공유 열람자 목록을 확인하여 이미 존재하는 열람자인지 확인
for (ApprovalShareReaderDto readerDto : createCommandInDto.getApprovalShareReaderList()) {
    // 데이터베이스에서 해당 결재 공유와 사용 중인 상태인 열람자를 조회합니다.
    Optional<ApprovalShareReader> existingReader = approvalShareReaderRepository
            .findByApprovalShareAndUserIdAndUseYn(approvalShare, readerDto.getUserId(), YesNo.Y);

    if (existingReader.isPresent()) {
        // 이미 존재하는 열람자라면 예외를 던지거나, 상황에 따라 처리합니다.
        throw new DuplicateReaderException("사용자 ID " + readerDto.getUserId() + "로 이미 존재하는 결재 공유 열람자가 있습니다");
    }
}

// 기존 로직: 결재 공유 열람자 화면 생성
List<ApprovalShareReaderView> approvalShareReaderViewList = createApprovalShareReaderViewList(createCommandInDto.getApprovalShareReaderList());
List<String> approvalShareReaderViewIdList = approvalShareReaderViewList.stream().map(ApprovalShareReaderView::getApprovalShareReaderViewId).collect(Collectors.toList());

// 기존 로직: 결재 공유 공유자 화면 생성
ApprovalShareSharerView approvalShareSharerView = createApprovalShareSharerView(createCommandInDto.getApprovalShareSharer());

// 기존 로직: 결재 공유 생성
ApprovalShare approvalShare = createApprovalShare(createCommandInDto.getParentApprovalLineId(), approvalShareReaderViewList, approvalShareSharerView, approvalLine);

// 결과 반환
return ApprovalShareCreateCommandOutDto.builder()
        .approvalShareId(approvalShare.getApprovalShareId())
        .approvalShareSharerViewId(approvalShareSharerView.getApprovalShareSharerViewId())
        .approvalShareReaderViewIdList(approvalShareReaderViewIdList)
        .build();

```
---> approvalShareReaderRepository.findByApprovalShareAndUserIdAndUseYn는 데이터베이스에서 결재 공유와 사용 중인 열람자를 조회하는 가상의 메서드입니다. 
