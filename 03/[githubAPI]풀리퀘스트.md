## github API로 pull request

일단 폼을 파고 내용을 입력하고 전송해야 하는데 베이스는 메인브랜치라고 치고 헤드를 알아야 함 사용자가 직접 입력해줘야?
아니면 브랜치들을 다 가져와서 select-option으로 띄우고 사용자가 선택하게 할 수 있음 -> 이게 깃허브 사이트 내의 방법

### Create a pull request

```js
// 옥토킷 버전

await octokit.request("POST /repos/{owner}/{repo}/pulls", {
  owner: "OWNER",
  repo: "REPO",
  title: "Amazing new feature",
  body: "Please pull these awesome changes in!",
  head: "octocat:new-feature",
  base: "master",
});
```

`head`가 내가 수정을 한 브랜치  
`base`는 말 그대로 베이스, 수정 내용이 병합될 브랜치(메인이나 디벨롭 같은)
