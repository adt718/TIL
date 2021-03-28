#足し算 JavaScript編
-2つの正の整数がa, bが入力されるのでaとbを足した数を出力してください。

-入力例1
1 1

-出力例1
2

-入力例2
0 99

-出力例2
99


#解答欄
process.stdin.resume();
process.stdin.setEncoding('utf8');
// 自分の得意な言語で
// Let's チャレンジ！！
var lines = [1+1];
var reader = require('readline').createInterface({
  input: process.stdin,
  output: process.stdout
});
reader.on('line', (line) => {
  lines.push(line);
});
reader.on('close', () => {
  console.log(lines[0]);
});
