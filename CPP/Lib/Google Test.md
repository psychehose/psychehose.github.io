
```
EXPECT_EQ(val1, val2);    // val1 == val2
EXPECT_NE(val1, val2);    // val1 != val2
EXPECT_LT(val1, val2);    // val1 < val2
EXPECT_LE(val1, val2);    // val1 <= val2
EXPECT_GT(val1, val2);    // val1 > val2
EXPECT_GE(val1, val2);    // val1 >= val2

EXPECT_TRUE(condition);    // condition이 true인지
EXPECT_FALSE(condition);   // condition이 false인지

EXPECT_STREQ(str1, str2);     // 문자열이 같은지
EXPECT_STRNE(str1, str2);     // 문자열이 다른지
EXPECT_STRCASEEQ(str1, str2); // 대소문자 무시하고 같은지
EXPECT_STRCASENE(str1, str2); // 대소문자 무시하고 다른지

EXPECT_FLOAT_EQ(val1, val2);   // float 거의 같은지
EXPECT_DOUBLE_EQ(val1, val2);  // double 거의 같은지
EXPECT_NEAR(val1, val2, abs_error); // 지정된 오차 범위 내인지
```
