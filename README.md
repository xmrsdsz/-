#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
praise.py — 专门夸奖人的小工具
用法示例:
  python praise.py "小明"
  python praise.py "小红" --count 3 --trait smart
  python praise.py --interactive
  python praise.py "团队" --save praise.txt

特点:
- 内置中文、英文夸奖模板
- 支持按特质（smart, kind, creative, funny, reliable）定向夸奖
- 支持交互模式和一次性多条生成
- 可保存输出到文件
"""

import argparse
import random
import sys
from datetime import datetime

COMPLIMENTS = {
    "default": [
        "亲爱的{name}，你真棒！每次和你在一起都感觉被治愈 🌟",
        "{name}，你的存在让世界更美好，谢谢你做的一切 😊",
        "{name}，你有一种让人安心的气质，继续做你自己 💪",
        "哇，{name}，你今天特别闪亮，像星星一样耀眼 ✨",
        "{name}，你的努力值得被看见，真心佩服你 👏",
    ],
    "smart": [
        "{name}，你的想法总是那么聪明又有条理，学到东西很多 💡",
        "{name}，你分析问题的方式太棒了，脑袋真灵光 🧠",
        "{name}，遇见你就像打开了新世界的窗口，受益匪浅 🔍",
    ],
    "kind": [
        "{name}，你的善良温暖了很多人，世界因为你更柔软 ❤️",
        "{name}，谢谢你总是在别人需要时伸出手，你是大家的光 🌈",
        "{name}，你注重细节、体贴入微，让人很放心 🤗",
    ],
    "creative": [
        "{name}，你的创意总是出人意料，令人耳目一新 🎨",
        "{name}，你把不可能变成可能，脑洞真的太大了 🚀",
        "{name}，看你做的东西就知道你有艺术家的灵魂 ✨",
    ],
    "funny": [
        "{name}，有你在场氛围就不一样了，幽默感满分 😂",
        "{name}，你总能用一句话化解尴尬，太会了 😄",
        "{name}，你的笑话是团队最珍贵的快乐来源 🎉",
    ],
    "reliable": [
        "{name}，你让人感觉很可靠，遇到你就放心了 🛡️",
        "{name}，有你的项目总能按时交付，真是坚强后盾 ✅",
        "{name}，你说到做到的样子很让人安心，太棒了 🔒",
    ],
    "english": [
        "{name}, you're amazing — your presence brightens the day ✨",
        "Hey {name}, your work is outstanding and truly inspiring 👏",
        "{name}, your kindness makes a big difference to people around you ❤️",
    ],
}

EMOJI_FOOTERS = [
    "🌟", "✨", "👍", "💖", "👏", "😊", "🎉", "🙌", "💪", "💫"
]


def build_parser():
    p = argparse.ArgumentParser(description="生成专门夸奖人的句子")
    p.add_argument("name", nargs="?", help="被夸奖者的名字或称呼（可选，交互模式可不写）")
    p.add_argument("--trait", choices=["smart", "kind", "creative", "funny", "reliable", "english"],
                   help="按特质生成更贴切的夸奖")
    p.add_argument("--count", type=int, default=1, help="生成夸奖句子的数量（默认1）")
    p.add_argument("--save", help="把结果保存到指定文件路径")
    p.add_argument("--interactive", action="store_true", help="进入交互模式")
    return p


def pick_compliments(name: str, trait: str = None, count: int = 1):
    pool = []
    if trait:
        pool.extend(COMPLIMENTS.get(trait, []))
    # Always include default and a few english if no trait
    pool.extend(COMPLIMENTS["default"])
    if not trait:
        pool.extend(COMPLIMENTS["english"])

    # Ensure name placeholder replaced
    results = []
    for _ in range(count):
        tmpl = random.choice(pool)
        footer = random.choice(EMOJI_FOOTERS)
        text = tmpl.format(name=name) + " " + footer
        results.append(text)
    return results


def interactive_mode():
    print("进入交互式夸奖模式。输入空行退出。")
    while True:
        try:
            name = input("\n请输入被夸奖者的名字（或按回车退出）: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n已退出。")
            break
        if not name:
            print("已退出交互模式。")
            break

        trait = input("想突出哪种特质？(smart/kind/creative/funny/reliable/english 或回车随机): ").strip()
        if trait == "":
            trait = None
        elif trait not in ("smart", "kind", "creative", "funny", "reliable", "english"):
            print("未知特质，使用随机或默认模板。")
            trait = None

        try:
            cnt = int(input("需要几条夸奖？(默认1): ").strip() or "1")
        except ValueError:
            cnt = 1

        results = pick_compliments(name, trait, cnt)
        for r in results:
            print("- " + r)


def main(argv):
    parser = build_parser()
    args = parser.parse_args(argv)

    if args.interactive:
        interactive_mode()
        return 0

    if not args.name:
        print("请提供被夸奖者的名字，或使用 --interactive 进入交互模式。")
        parser.print_help()
        return 1

    results = pick_compliments(args.name, args.trait, max(1, args.count))
    out_text = "\n".join(results)

    # Optionally save
    if args.save:
        try:
            with open(args.save, "a", encoding="utf-8") as f:
                f.write(f"\n# Generated on {datetime.now().isoformat()}\n")
                f.write(out_text + "\n")
            print(f"已保存到 {args.save}")
        except Exception as e:
            print("保存文件失败：", e)

    print(out_text)
    return 0


if __name__ == "__main__":
    raise SystemExit(main(sys.argv[1:]))
