#掛け算 (paizaランク D 相当)に挑戦
-問題
-2つの正の整数a, bが改行区切りで入力されるのでaとbを掛け算した数値を出力してください。

##入力例
入力例1
2
3

出力例1
6

入力例2
0
99

出力例2
0

##解答例

process.stdin.resume();
process.stdin.setEncoding('utf8');
// 自分の得意な言語で
// Let's チャレンジ！！
var lines = [2*3];
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
