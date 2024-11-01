---
permalink: /infra/postquantum
layout: single
title: "Post Quantum Encryption"
---

## 概要

> ポスト量子暗号 (ポストりょうしあんごう、英:Post-quantum cryptography、略してPQC)とは、量子コンピュータによる暗号解読に対して安全だと考えられる暗号アルゴリズム (主に公開鍵暗号アルゴリズム) のことである。現在よく使われているアルゴリズムの問題は、そのセキュリティーが素因数分解、離散対数、楕円曲線暗号という３つの数学的な難題に依拠していることにある。 これらの問題はすべて、十分に強力な量子コンピュータとショアのアルゴリズム（英語版）[1][2]や、それよりも高速で必要とする量子ビットも少ないアルゴリズム[3]を用いることで容易に解くことができる。

> <cite><a href="https://ja.wikipedia.org/wiki/%E3%83%9D%E3%82%B9%E3%83%88%E9%87%8F%E5%AD%90%E6%9A%97%E5%8F%B7">Wikipedia</a></cite>

## 最近出た記事とその概要

[NIST Releases First 3 Finalized Post-Quantum Encryption Standards](https://www.nist.gov/news-events/news/2024/08/nist-releases-first-3-finalized-post-quantum-encryption-standards)

Gigazine の解説がわかりやすかった

[アメリカ国立標準技術研究所が3つのポスト量子暗号の最終標準「ML-KEM」「ML-DSA」「SLH-DSA」をリリース](https://gigazine.net/news/20240814-nist-releases-post-quantum-encryption-standards/)


> 1つめの「ML-KEM」は、Crystal Kyberから派生したもので、保護されたWebサイトへのアクセスなど一般的な暗号化のために使用される主要なカプセル化メカニズムです。
> 2つめの「ML-DSA」は、Crystal Dilithiumから派生したもので、汎用デジタル署名プロトコル用の格子ベースのアルゴリズムだとのこと。
> 3つめの「SLH-DSA」は、SPHINCS+から派生したもので、ステートレスハッシュベースのデジタル署名方式です。
> それぞれ、商務長官によって連邦情報処理基準(FIPS)の承認を受け「FIPS 203」「FIPS 204」「FIPS 205」の番号が与えられています。
