---
layout: default
---
Strategy: read Mathworld about the Sum of Squares Function, rely on
Lagrange having proved any integer can be represented by at most four
squares.  Get the lists of two- and four-square sums from the Online
Encyclopedia of Integer Sequences.

```python
import math
import sets
def answer(n):
    # try one square, see if we're lucky
    root = math.sqrt(n)
    if root == int(root):
        return 1

    # The sets of integers and how many squares are required
    # to represent them is available at:
    # http://oeis.org/wiki/Index_to_OEIS:_Section_Su#ssq
    # Doing your own math is like doing your own crypto.

    two_squares = sets.ImmutableSet(
        [0, 1, 2, 4, 5, 8, 9, 10, 13, 16,
        17, 18, 20, 25, 26, 29, 32, 34, 36, 37,
        40, 41, 45, 49, 50, 52, 53, 58, 61, 64,
        65, 68, 72, 73, 74, 80, 81, 82, 85, 89,
        90, 97, 98, 100, 101, 104, 106, 109, 113, 116,
        117, 121, 122, 125, 128, 130, 136, 137, 144, 145,
        146, 148, 149, 153, 157, 160, 162, 164, 169, 170,
        173, 178, 180, 181, 185, 193, 194, 196, 197, 200,
        202, 205, 208, 212, 218, 221, 225, 226, 229, 232,
        233, 234, 241, 242, 244, 245, 250, 256, 257, 260,
        261, 265, 269, 272, 274, 277, 281, 288, 289, 290,
        292, 293, 296, 298, 305, 306, 313, 314, 317, 320,
        324, 325, 328, 333, 337, 338, 340, 346, 349, 353,
        356, 360, 361, 362, 365, 369, 370, 373, 377, 386,
        388, 389, 392, 394, 397, 400, 401, 404, 405, 409,
        410, 416, 421, 424, 425, 433, 436, 441, 442, 445,
        449, 450, 452, 457, 458, 461, 464, 466, 468, 477,
        481, 482, 484, 485, 488, 490, 493, 500, 505, 509,
        512, 514, 520, 521, 522, 529, 530, 533, 538, 541,
        544, 545, 548, 549, 554, 557, 562, 565, 569, 576,
        577, 578, 580, 584, 585, 586, 592, 593, 596, 601,
        605, 610, 612, 613, 617, 625, 626, 628, 629, 634,
        637, 640, 641, 648, 650, 653, 656, 657, 661, 666,
        673, 674, 676, 677, 680, 685, 689, 692, 697, 698,
        701, 706, 709, 712, 720, 722, 724, 725, 729, 730,
        733, 738, 740, 745, 746, 754, 757, 761, 765, 769,
        772, 773, 776, 778, 784, 785, 788, 793, 794, 797,
        800, 801, 802, 808, 809, 810, 818, 820, 821, 829,
        832, 833, 841, 842, 845, 848, 850, 853, 857, 865,
        866, 872, 873, 877, 881, 882, 884, 890, 898, 900,
        901, 904, 905, 909, 914, 916, 922, 925, 928, 929,
        932, 936, 937, 941, 949, 953, 954, 961, 962, 964,
        965, 968, 970, 976, 977, 980, 981, 985, 986, 997,
        1000, 1009, 1010, 1013, 1017, 1018, 1021, 1024, 1025, 1028,
        1033, 1037, 1040, 1042, 1044, 1049, 1053, 1058, 1060, 1061,
        1066, 1069, 1073, 1076, 1082, 1088, 1089, 1090, 1093, 1096,
        1097, 1098, 1105, 1108, 1109, 1114, 1117, 1124, 1125, 1129,
        1130, 1138, 1145, 1152, 1153, 1154, 1156, 1157, 1160, 1165,
        1168, 1170, 1172, 1181, 1184, 1186, 1189, 1192, 1193, 1201,
        1202, 1205, 1210, 1213, 1217, 1220, 1224, 1225, 1226, 1229,
        1233, 1234, 1237, 1241, 1249, 1250, 1252, 1256, 1258, 1261,
        1268, 1274, 1277, 1280, 1282, 1285, 1289, 1296, 1297, 1300,
        1301, 1305, 1306, 1312, 1313, 1314, 1321, 1322, 1325, 1332,
        1341, 1345, 1346, 1348, 1352, 1354, 1360, 1361, 1369, 1370,
        1373, 1377, 1378, 1381, 1384, 1385, 1394, 1396, 1402, 1405,
        1409, 1412, 1413, 1417, 1418, 1421, 1424, 1429, 1433, 1440,
        1444, 1445, 1448, 1450, 1453, 1458, 1460, 1465, 1466, 1469,
        1476, 1480, 1481, 1489, 1490, 1492, 1493, 1508, 1513, 1514,
        1517, 1521, 1522, 1525, 1530, 1537, 1538, 1544, 1546, 1549,
        1552, 1553, 1556, 1557, 1565, 1568, 1570, 1573, 1576, 1585,
        1586, 1588, 1594, 1597, 1600, 1601, 1602, 1604, 1609, 1613,
        1616, 1618, 1620, 1621, 1625, 1629, 1636, 1637, 1640, 1642,
        1649, 1657, 1658, 1664, 1665, 1666, 1669, 1681, 1682, 1684,
        1685, 1690, 1693, 1696, 1697, 1700, 1706, 1709, 1714, 1717,
        1721, 1730, 1732, 1733, 1737, 1741, 1744, 1745, 1746, 1753,
        1754, 1762, 1764, 1765, 1768, 1769, 1773, 1777, 1780, 1781,
        1789, 1796, 1800, 1801, 1802, 1805, 1808, 1810, 1813, 1818,
        1825, 1828, 1832, 1844, 1845, 1849, 1850, 1853, 1856, 1858,
        1861, 1864, 1865, 1872, 1873, 1874, 1877, 1882, 1885, 1889,
        1898, 1901, 1906, 1908, 1913, 1921, 1922, 1924, 1928, 1930,
        1933, 1936, 1937, 1940, 1945, 1949, 1952, 1954, 1960, 1961,
        1962, 1970, 1972, 1973, 1985, 1989, 1993, 1994, 1997, 2000,
        2005, 2009, 2017, 2018, 2020, 2025, 2026, 2029, 2034, 2036,
        2041, 2042, 2045, 2048, 2050, 2053, 2056, 2057, 2061, 2066,
        2069, 2074, 2080, 2081, 2084, 2088, 2089, 2097, 2098, 2105,
        2106, 2113, 2116, 2117, 2120, 2122, 2125, 2129, 2132, 2137,
        2138, 2141, 2146, 2152, 2153, 2161, 2164, 2165, 2169, 2173,
        2176, 2178, 2180, 2186, 2192, 2194, 2196, 2197, 2205, 2209,
        2210, 2213, 2216, 2218, 2221, 2225, 2228, 2234, 2237, 2245,
        2248, 2249, 2250, 2257, 2258, 2260, 2269, 2273, 2276, 2281,
        2285, 2290, 2293, 2297, 2304, 2305, 2306, 2308, 2309, 2312,
        2313, 2314, 2320, 2329, 2330, 2333, 2336, 2340, 2341, 2344,
        2349, 2353, 2357, 2362, 2368, 2372, 2377, 2378, 2381, 2384,
        2385, 2386, 2389, 2393, 2401, 2402, 2404, 2405, 2410, 2417,
        2420, 2421, 2425, 2426, 2434, 2437, 2440, 2441, 2448, 2450,
        2452, 2458, 2465, 2466, 2468, 2473, 2474, 2477, 2482, 2493,
        2498, 2500, 2501, 2504, 2509, 2512, 2516, 2521, 2522, 2525,
        2529, 2533, 2536, 2545, 2548, 2549, 2554, 2557, 2560, 2561,
        2564, 2570, 2578, 2581, 2592, 2593, 2594, 2597, 2600, 2601,
        2602, 2605, 2609, 2610, 2612, 2617, 2621, 2624, 2626, 2628,
        2633, 2637, 2642, 2644, 2645, 2650, 2657, 2664, 2665, 2669,
        2677, 2682, 2689, 2690, 2692, 2693, 2696, 2701, 2704, 2705,
        2708, 2713, 2720, 2722, 2725, 2729, 2738, 2740, 2741, 2745,
        2746, 2749, 2753, 2754, 2756, 2762, 2768, 2770, 2777, 2785,
        2788, 2789, 2792, 2797, 2801, 2804, 2809, 2810, 2813, 2817,
        2818, 2824, 2825, 2826, 2833, 2834, 2836, 2837, 2842, 2845,
        2848, 2853, 2857, 2858, 2861, 2866, 2873, 2880, 2885, 2888,
        2890, 2896, 2897, 2900, 2906, 2909, 2916, 2917, 2920, 2925,
        2929, 2930, 2932, 2938, 2941, 2952, 2953, 2957, 2960, 2962,
        2965, 2969, 2977, 2978, 2980, 2984, 2986, 2989, 2993, 2997,
        3001, 3005, 3016, 3025, 3026, 3028, 3029, 3033, 3034, 3037,
        3041, 3042, 3044, 3049, 3050, 3060, 3061, 3065, 3074, 3076,
        3077, 3085, 3088, 3089, 3092, 3098, 3104, 3106, 3109, 3112,
        3114, 3121, 3125, 3130, 3133, 3136, 3137, 3140, 3141, 3145,
        3146, 3152, 3161, 3169, 3170, 3172, 3176, 3177, 3181, 3185,
        3188, 3194, 3200, 3202, 3204, 3205, 3208, 3209, 3217, 3218,
        3221, 3226, 3229, 3232, 3233, 3236, 3240, 3242, 3249, 3250,
        3253, 3257, 3258, 3265, 3272, 3274, 3277, 3280, 3281, 3284,
        3285, 3293, 3298, 3301, 3305, 3313, 3314, 3316, 3321, 3328,
        3329, 3330, 3332, 3338, 3341, 3349, 3357, 3361, 3362, 3364,
        3365, 3368, 3370, 3373, 3380, 3385, 3386, 3389, 3392, 3393,
        3394, 3400, 3412, 3413, 3418, 3425, 3428, 3433, 3434, 3442,
        3445, 3449, 3457, 3460, 3461, 3464, 3466, 3469, 3474, 3481,
        3482, 3485, 3488, 3490, 3492, 3497, 3501, 3505, 3506, 3508,
        3509, 3517, 3524, 3528, 3529, 3530, 3533, 3536, 3538, 3541,
        3545, 3546, 3554, 3557, 3560, 3562, 3573, 3577, 3578, 3581,
        3589, 3592, 3593, 3600, 3601, 3602, 3604, 3609, 3610, 3613,
        3616, 3617, 3620, 3625, 3626, 3636, 3637, 3645, 3649, 3650,
        3653, 3656, 3664, 3665, 3673, 3677, 3681, 3688, 3690, 3697,
        3698, 3700, 3701, 3706, 3709, 3712, 3716, 3721, 3722, 3725,
        3728, 3730, 3733, 3737, 3744, 3746, 3748, 3754, 3757, 3761,
        3764, 3769, 3770, 3778, 3785, 3789, 3793, 3796, 3797, 3802,
        3805, 3809, 3812, 3816, 3821, 3825, 3826, 3833, 3842, 3844,
        3845, 3848, 3853, 3856, 3860, 3865, 3866, 3869, 3872, 3874,
        3877, 3880, 3881, 3889, 3890, 3893, 3897, 3898, 3904, 3908,
        3917, 3920, 3922, 3924, 3925, 3929, 3940, 3944, 3946, 3961,
        3965, 3969, 3970, 3973, 3977, 3978, 3985, 3986, 3988, 3989,
        3994, 4000, 4001, 4005, 4010, 4013, 4018, 4021, 4033, 4034,
        4036, 4040, 4041, 4045, 4049, 4050, 4052, 4057, 4058, 4068,
        4069, 4072, 4073, 4082, 4084, 4090, 4093, 4096, 4097, 4100,
        4105, 4106, 4112, 4113, 4114, 4121, 4122, 4129, 4132, 4133,
        4138, 4141, 4145, 4148, 4149, 4153, 4157, 4160, 4162, 4165,
        4168, 4176, 4177, 4178, 4181, 4194, 4196, 4201, 4205, 4210,
        4212, 4217, 4225, 4226, 4229, 4232, 4234, 4240, 4241, 4244,
        4250, 4253, 4258, 4261, 4264, 4265, 4273, 4274, 4276, 4282,
        4285, 4289, 4292, 4293, 4297, 4304, 4306, 4321, 4322, 4325,
        4328, 4329, 4330, 4337, 4338, 4346, 4349, 4352, 4356, 4357,
        4360, 4361, 4365, 4369, 4372, 4373, 4381, 4384, 4385, 4388,
        4392, 4394, 4397, 4405, 4409, 4410, 4418, 4420, 4421, 4426,
        4432, 4436, 4437, 4441, 4442, 4450, 4453, 4456, 4457, 4468,
        4469, 4474, 4477, 4481, 4489, 4490, 4493, 4496, 4498, 4500,
        4505, 4513, 4514, 4516, 4517, 4520, 4525, 4537, 4538, 4545,
        4546, 4549, 4552, 4553, 4561, 4562, 4570, 4573, 4580, 4581,
        4586, 4589, 4594, 4597, 4608, 4610, 4612, 4616, 4618, 4621,
        4624, 4625, 4626, 4628, 4633, 4637, 4640, 4645, 4649, 4657,
        4658, 4660, 4666, 4672, 4673, 4680, 4682, 4685, 4688, 4689,
        4693, 4698, 4705, 4706, 4709, 4714, 4717, 4721, 4724, 4729,
        4733, 4736, 4744, 4745, 4753, 4754, 4756, 4761, 4762, 4765,
        4768, 4770, 4772, 4777, 4778, 4786, 4789, 4793, 4797, 4801,
        4802, 4804, 4805, 4808, 4810, 4813, 4817, 4820, 4825, 4834,
        4840, 4842, 4849, 4850, 4852, 4861, 4868, 4869, 4874, 4877,
        4880, 4882, 4885, 4889, 4896, 4900, 4901, 4904, 4905, 4909,
        4913, 4916, 4925, 4930, 4932, 4933, 4936, 4937, 4941, 4946,
        4948, 4949, 4954, 4957, 4961, 4964, 4969, 4973, 4981, 4985,
        4986, 4993, 4996, 5000, 5002, 5008, 5009, 5013, 5017, 5018,
        5021, 5024, 5032, 5041, 5042, 5044, 5045, 5050, 5057, 5058,
        5065, 5066, 5069, 5072, 5077, 5081, 5085, 5090, 5096, 5098,
        5101, 5105, 5108, 5113, 5114, 5120, 5121, 5122, 5125, 5128,
        5140, 5141, 5153, 5156, 5161, 5162, 5165, 5184, 5185, 5186,
        5188, 5189, 5193, 5194, 5197, 5200, 5202, 5204, 5209, 5210,
        5213, 5218, 5220, 5224, 5233, 5234, 5237, 5242, 5245, 5248,
        5249, 5252, 5256, 5261, 5265, 5266, 5273, 5274, 5281, 5284,
        5288, 5290, 5297, 5300, 5305, 5309, 5314, 5317, 5321, 5328,
        5329, 5330, 5333, 5337, 5338, 5341, 5345, 5353, 5354, 5364,
        5365, 5378, 5380, 5381, 5384, 5386, 5389, 5392, 5393, 5402,
        5408, 5409, 5410, 5413, 5416, 5417, 5426, 5429, 5437, 5440,
        5441, 5444, 5445, 5449, 5450, 5458, 5465, 5473, 5476, 5477,
        5480, 5482, 5485, 5490, 5492, 5498, 5501, 5506, 5508, 5512,
        5513, 5517, 5521, 5524, 5525, 5536, 5537, 5540, 5545, 5553,
        5554, 5557, 5569, 5570, 5573, 5576, 5578, 5581, 5584, 5585,
        5594, 5597, 5602, 5608, 5617, 5618, 5620, 5625, 5626, 5629,
        5634, 5636, 5641, 5645, 5648, 5650, 5652, 5653, 5657, 5661,
        5666, 5668, 5669, 5672, 5674, 5684, 5689, 5690, 5693, 5696,
        5701, 5706, 5713, 5714, 5716, 5717, 5722, 5725, 5729, 5732,
        5733, 5737, 5741, 5746, 5749, 5760, 5765, 5769, 5770, 5776,
        5777, 5780, 5785, 5792, 5794, 5800, 5801, 5809, 5812, 5813,
        5818, 5821, 5825, 5832, 5834, 5837, 5840, 5849, 5850, 5857,
        5858, 5860, 5861, 5864, 5869, 5876, 5877, 5881, 5882, 5897,
        5904, 5905, 5906, 5913, 5914, 5917, 5920, 5924, 5929, 5930,
        5933, 5938, 5941, 5945, 5949, 5953, 5954, 5956, 5960, 5965,
        5968, 5972, 5978, 5981, 5986, 5989, 5993, 5994, 6001, 6002,
        6005, 6010, 6025, 6029, 6032, 6037, 6050, 6052, 6053, 6056,
        6057, 6058, 6065, 6066, 6068, 6073, 6074, 6082, 6084, 6085,
        6088, 6089, 6093, 6098, 6100, 6101, 6109, 6113, 6120, 6121,
        6122, 6125, 6130, 6133, 6137, 6145, 6148, 6152, 6154, 6161,
        6165, 6170, 6173, 6176, 6178, 6184, 6185, 6196, 6197, 6201,
        6205, 6208, 6212, 6217, 6218, 6221, 6224, 6228, 6229, 6241,
        6242, 6245, 6250, 6253, 6257, 6260, 6266, 6269, 6272, 6273,
        6274, 6277, 6280, 6282, 6290, 6292, 6301, 6304, 6305, 6309,
        6317, 6322, 6329, 6337, 6338, 6340, 6341, 6344, 6352, 6353,
        6354, 6361, 6362, 6370, 6373, 6376, 6381, 6385, 6388, 6389,
        6397, 6400, 6401, 6404, 6408, 6409, 6410, 6413, 6416, 6418,
        6421, 6425, 6434, 6436, 6437, 6442, 6445, 6449, 6452, 6458,
        6464, 6466, 6469, 6472, 6473, 6480, 6481, 6484, 6485, 6497,
        6498, 6500, 6505, 6506, 6514, 6516, 6521, 6525, 6529, 6530,
        6544, 6548, 6553, 6554, 6560, 6561, 6562, 6565, 6568, 6569,
        6570, 6577, 6581, 6586, 6596, 6597, 6602, 6605, 6610, 6613,
        6617, 6625, 6626, 6628, 6632, 6637, 6641, 6642, 6649, 6653,
        6656, 6658, 6660, 6661, 6664, 6673, 6676, 6682, 6689, 6697,
        6698, 6701, 6705, 6709, 6713, 6714, 6722, 6724, 6725, 6728,
        6730, 6733, 6736, 6737, 6740, 6746, 6749, 6757, 6760, 6761,
        6770, 6772, 6773, 6778, 6781, 6784, 6786, 6788, 6793, 6800,
        6805, 6813, 6817, 6824, 6826, 6829, 6833, 6836, 6841, 6845,
        6849, 6850, 6856, 6857, 6865, 6866, 6868, 6869, 6877, 6884,
        6885, 6889, 6890, 6893, 6898, 6905, 6914, 6917, 6920, 6921,
        6922, 6925, 6928, 6929, 6932, 6938, 6948, 6949, 6953, 6957,
        6961, 6962, 6964, 6970, 6976, 6977, 6980, 6984, 6989, 6994,
        6997, 7001, 7002, 7010, 7012, 7013, 7016, 7018, 7025, 7033,
        7034, 7045, 7048, 7056, 7057, 7058, 7060, 7065, 7066, 7069,
        7072, 7076, 7081, 7082, 7085, 7090, 7092, 7093, 7105, 7108,
        7109, 7114, 7120, 7121, 7124, 7129, 7137, 7141, 7145, 7146,
        7154, 7156, 7157, 7162, 7165, 7173, 7177, 7178, 7184, 7186,
        7193, 7200, 7202, 7204, 7208, 7209, 7213, 7218, 7220, 7225,
        7226, 7229, 7232, 7234, 7237, 7240, 7241, 7250, 7252, 7253,
        7261, 7265, 7272, 7274, 7281, 7289, 7290, 7297, 7298, 7300,
        7301, 7306, 7309, 7312, 7321, 7325, 7328, 7330, 7333, 7345,
        7346, 7349, 7354, 7361, 7362, 7369, 7373, 7376, 7380, 7381,
        7389, 7393, 7394, 7396, 7397, 7400, 7402, 7405, 7412, 7417,
        7418, 7421, 7424, 7432, 7433, 7442, 7444, 7445, 7450, 7453,
        7456, 7457, 7460, 7461, 7465, 7466, 7474, 7477, 7481, 7488,
        7489, 7492, 7496, 7497, 7501, 7508, 7514, 7517, 7522, 7528,
        7529, 7537, 7538, 7540, 7541, 7549, 7556, 7561, 7565, 7569,
        7570, 7573, 7577, 7578, 7585, 7586, 7589, 7592, 7594, 7604,
        7605, 7610, 7618, 7621, 7624, 7625, 7632, 7633, 7642, 7649,
        7650, 7652, 7666, 7669, 7673, 7677, 7681, 7684, 7685, 7688,
        7690, 7693, 7696, 7706, 7709, 7712, 7713, 7717, 7720, 7730,
        7732, 7738, 7741, 7744, 7745, 7748, 7753, 7754, 7757, 7760,
        7762, 7765, 7769, 7778, 7780, 7785, 7786, 7789, 7793, 7794,
        7796, 7801, 7808, 7813, 7816, 7817, 7825, 7829, 7834, 7837,
        7840, 7841, 7844, 7848, 7850, 7853, 7857, 7858, 7865, 7873,
        7877, 7880, 7888, 7892, 7893, 7897, 7901, 7913, 7921, 7922,
        7925, 7929, 7930, 7933, 7937, 7938, 7940, 7946, 7949, 7954,
        7956, 7957, 7969, 7970, 7972, 7976, 7978, 7985, 7988, 7993,
        8000, 8002, 8005, 8009, 8010, 8017, 8020, 8021, 8026, 8033,
        8036, 8042, 8045, 8053, 8065, 8066, 8068, 8069, 8072, 8077,
        8080, 8081, 8082, 8089, 8090, 8093, 8098, 8100, 8101, 8104,
        8105, 8109, 8114, 8116, 8117, 8125, 8136, 8138, 8144, 8145,
        8146, 8149, 8161, 8164, 8168, 8177, 8180, 8181, 8185, 8186,
        8192, 8194, 8200, 8209, 8210, 8212, 8221, 8224, 8226, 8228,
        8233, 8237, 8242, 8244, 8245, 8249, 8258, 8264, 8266, 8269,
        8273, 8276, 8281, 8282, 8285, 8290, 8293, 8296, 8297, 8298,
        8306, 8314, 8317, 8320, 8321, 8324, 8325, 8329, 8330, 8333,
        8336, 8345, 8352, 8353, 8354, 8356, 8357, 8361, 8362, 8369,
        8377, 8381, 8388, 8389, 8392, 8402, 8405, 8410, 8420, 8424,
        8425, 8429, 8433, 8434, 8450, 8452, 8458, 8461, 8464, 8465,
        8468, 8469, 8473, 8477, 8480, 8482, 8485, 8488, 8489, 8497,
        8500, 8501, 8506, 8513, 8516, 8521, 8522, 8528, 8530, 8537,
        8541, 8545, 8546, 8548, 8552, 8564, 8570, 8573, 8577, 8578,
        8581, 8584, 8585, 8586, 8593, 8594, 8597, 8605, 8608, 8609,
        8612, 8621, 8629, 8633, 8641, 8642, 8644, 8649, 8650, 8653,
        8656, 8658, 8660, 8665, 8669, 8674, 8676, 8677, 8681, 8685,
        8689, 8692, 8693, 8698, 8704, 8705, 8712, 8713, 8714, 8720,
        8722, 8725, 8730, 8737, 8738, 8741, 8744, 8746, 8749, 8753,
        8761, 8762, 8765, 8768, 8770, 8776, 8784, 8788, 8793, 8794,
        8801, 8810, 8818, 8820, 8821, 8825, 8829, 8833, 8836, 8837,
        8840, 8842, 8845, 8849, 8852, 8857, 8861, 8864, 8865, 8869,
        8872, 8874, 8882, 8884, 8885, 8893, 8900, 8905, 8906, 8912,
        8914, 8917, 8929, 8933, 8936, 8938, 8941, 8945, 8948, 8954,
        8957, 8962, 8969, 8973, 8978, 8980, 8986, 8989, 8992, 8993,
        8996, 9000, 9001, 9005, 9010, 9013, 9025, 9026, 9028, 9029,
        9032, 9034, 9040, 9041, 9049, 9050, 9061, 9065, 9074, 9076,
        9077, 9081, 9089, 9090, 9092, 9098, 9104, 9106, 9109, 9113,
        9117, 9122, 9124, 9125, 9133, 9137, 9140, 9146, 9153, 9157,
        9160, 9161, 9162, 9169, 9172, 9173, 9178, 9181, 9188, 9189,
        9193, 9194, 9197, 9209, 9216, 9217, 9220, 9221, 9224, 9225,
        9232, 9236, 9241, 9242, 9245, 9248, 9250, 9252, 9256, 9257,
        9265, 9266, 9274, 9277, 9280, 9281, 9290, 9293, 9297, 9298,
        9305, 9314, 9316, 9320, 9325, 9332, 9333, 9337, 9341, 9344,
        9346, 9349, 9360, 9364, 9365, 9370, 9376, 9377, 9378, 9385,
        9386, 9389, 9396, 9397, 9409, 9410, 9412, 9413, 9418, 9421,
        9425, 9428, 9433, 9434, 9437, 9441, 9442, 9445, 9448, 9457,
        9458, 9461, 9466, 9469, 9472, 9473, 9477, 9488, 9490, 9497,
        9505, 9506, 9508, 9509, 9512, 9521, 9522, 9524, 9529, 9530,
        9533, 9536, 9540, 9544, 9549, 9553, 9554, 9556, 9565, 9572,
        9577, 9578, 9586, 9593, 9594, 9601, 9602, 9604, 9605, 9608,
        9610, 9613, 9616, 9620, 9621, 9626, 9629, 9634, 9640, 9649,
        9650, 9653, 9657, 9661, 9665, 9668, 9673, 9677, 9680, 9684,
        9685, 9689, 9697, 9698, 9700, 9701, 9704, 9721, 9722, 9725,
        9733, 9736, 9738, 9745, 9748, 9749, 9754, 9760, 9764, 9769,
        9770, 9773, 9778, 9781, 9792, 9797, 9800, 9801, 9802, 9805,
        9808, 9809, 9810, 9817, 9818, 9826, 9829, 9832, 9833, 9837,
        9841, 9850, 9857, 9860, 9864, 9865, 9866, 9872, 9873, 9874,
        9881, 9882, 9892, 9893, 9896, 9898, 9901, 9908, 9914, 9922,
        9925, 9928, 9929, 9938, 9941, 9945, 9946, 9949, 9953, 9962,
        9965, 9970, 9972, 9973, 9981, 9985, 9986, 9992, 9997, 10000])


    four_squares = sets.ImmutableSet(
        [7, 15, 23, 28, 31, 39, 47, 55, 60, 63, 71, 79, 87,
        92, 95, 103, 111, 112, 119, 124, 127, 135, 143, 151,
        156, 159, 167, 175, 183, 188, 191, 199, 207, 215,
        220, 223, 231, 239, 240, 247, 252, 255, 263, 271,
        279, 284, 287, 295, 303, 311, 316, 319, 327, 335,
        343, 348, 351, 359, 367, 368, 375, 380, 383, 391,
        399, 407, 412, 415, 423, 431, 439, 444, 447, 448,
        455, 463, 471, 476, 479, 487, 495, 496, 503, 508,
        511, 519, 527, 535, 540, 543, 551, 559, 567, 572,
        575, 583, 591, 599, 604, 607, 615, 623, 624, 631,
        636, 639, 647, 655, 663, 668, 671, 679, 687, 695,
        700, 703, 711, 719, 727, 732, 735, 743, 751, 752,
        759, 764, 767, 775, 783, 791, 796, 799, 807, 815,
        823, 828, 831, 839, 847, 855, 860, 863, 871, 879,
        880, 887, 892, 895, 903, 911, 919, 924, 927, 935,
        943, 951, 956, 959, 960, 967, 975, 983, 988, 991,
        999, 1007, 1008, 1015, 1020, 1023, 1031, 1039, 1047, 1052,
        1055, 1063, 1071, 1079, 1084, 1087, 1095, 1103, 1111, 1116,
        1119, 1127, 1135, 1136, 1143, 1148, 1151, 1159, 1167, 1175,
        1180, 1183, 1191, 1199, 1207, 1212, 1215, 1223, 1231, 1239,
        1244, 1247, 1255, 1263, 1264, 1271, 1276, 1279, 1287, 1295,
        1303, 1308, 1311, 1319, 1327, 1335, 1340, 1343, 1351, 1359,
        1367, 1372, 1375, 1383, 1391, 1392, 1399, 1404, 1407, 1415,
        1423, 1431, 1436, 1439, 1447, 1455, 1463, 1468, 1471, 1472,
        1479, 1487, 1495, 1500, 1503, 1511, 1519, 1520, 1527, 1532,
        1535, 1543, 1551, 1559, 1564, 1567, 1575, 1583, 1591, 1596,
        1599, 1607, 1615, 1623, 1628, 1631, 1639, 1647, 1648, 1655,
        1660, 1663, 1671, 1679, 1687, 1692, 1695, 1703, 1711, 1719,
        1724, 1727, 1735, 1743, 1751, 1756, 1759, 1767, 1775, 1776,
        1783, 1788, 1791, 1792, 1799, 1807, 1815, 1820, 1823, 1831,
        1839, 1847, 1852, 1855, 1863, 1871, 1879, 1884, 1887, 1895,
        1903, 1904, 1911, 1916, 1919, 1927, 1935, 1943, 1948, 1951,
        1959, 1967, 1975, 1980, 1983, 1984, 1991, 1999, 2007, 2012,
        2015, 2023, 2031, 2032, 2039, 2044, 2047, 2055, 2063, 2071,
        2076, 2079, 2087, 2095, 2103, 2108, 2111, 2119, 2127, 2135,
        2140, 2143, 2151, 2159, 2160, 2167, 2172, 2175, 2183, 2191,
        2199, 2204, 2207, 2215, 2223, 2231, 2236, 2239, 2247, 2255,
        2263, 2268, 2271, 2279, 2287, 2288, 2295, 2300, 2303, 2311,
        2319, 2327, 2332, 2335, 2343, 2351, 2359, 2364, 2367, 2375,
        2383, 2391, 2396, 2399, 2407, 2415, 2416, 2423, 2428, 2431,
        2439, 2447, 2455, 2460, 2463, 2471, 2479, 2487, 2492, 2495,
        2496, 2503, 2511, 2519, 2524, 2527, 2535, 2543, 2544, 2551,
        2556, 2559, 2567, 2575, 2583, 2588, 2591, 2599, 2607, 2615,
        2620, 2623, 2631, 2639, 2647, 2652, 2655, 2663, 2671, 2672,
        2679, 2684, 2687, 2695, 2703, 2711, 2716, 2719, 2727, 2735,
        2743, 2748, 2751, 2759, 2767, 2775, 2780, 2783, 2791, 2799,
        2800, 2807, 2812, 2815, 2823, 2831, 2839, 2844, 2847, 2855,
        2863, 2871, 2876, 2879, 2887, 2895, 2903, 2908, 2911, 2919,
        2927, 2928, 2935, 2940, 2943, 2951, 2959, 2967, 2972, 2975,
        2983, 2991, 2999, 3004, 3007, 3008, 3015, 3023, 3031, 3036,
        3039, 3047, 3055, 3056, 3063, 3068, 3071, 3079, 3087, 3095,
        3100, 3103, 3111, 3119, 3127, 3132, 3135, 3143, 3151, 3159,
        3164, 3167, 3175, 3183, 3184, 3191, 3196, 3199, 3207, 3215,
        3223, 3228, 3231, 3239, 3247, 3255, 3260, 3263, 3271, 3279,
        3287, 3292, 3295, 3303, 3311, 3312, 3319, 3324, 3327, 3335,
        3343, 3351, 3356, 3359, 3367, 3375, 3383, 3388, 3391, 3399,
        3407, 3415, 3420, 3423, 3431, 3439, 3440, 3447, 3452, 3455,
        3463, 3471, 3479, 3484, 3487, 3495, 3503, 3511, 3516, 3519,
        3520, 3527, 3535, 3543, 3548, 3551, 3559, 3567, 3568, 3575,
        3580, 3583, 3591, 3599, 3607, 3612, 3615, 3623, 3631, 3639,
        3644, 3647, 3655, 3663, 3671, 3676, 3679, 3687, 3695, 3696,
        3703, 3708, 3711, 3719, 3727, 3735, 3740, 3743, 3751, 3759,
        3767, 3772, 3775, 3783, 3791, 3799, 3804, 3807, 3815, 3823,
        3824, 3831, 3836, 3839, 3840, 3847, 3855, 3863, 3868, 3871,
        3879, 3887, 3895, 3900, 3903, 3911, 3919, 3927, 3932, 3935,
        3943, 3951, 3952, 3959, 3964, 3967, 3975, 3983, 3991, 3996,
        3999, 4007, 4015, 4023, 4028, 4031, 4032, 4039, 4047, 4055,
        4060, 4063, 4071, 4079, 4080, 4087, 4092, 4095, 4103, 4111,
        4119, 4124, 4127, 4135, 4143, 4151, 4156, 4159, 4167, 4175,
        4183, 4188, 4191, 4199, 4207, 4208, 4215, 4220, 4223, 4231,
        4239, 4247, 4252, 4255, 4263, 4271, 4279, 4284, 4287, 4295,
        4303, 4311, 4316, 4319, 4327, 4335, 4336, 4343, 4348, 4351,
        4359, 4367, 4375, 4380, 4383, 4391, 4399, 4407, 4412, 4415,
        4423, 4431, 4439, 4444, 4447, 4455, 4463, 4464, 4471, 4476,
        4479, 4487, 4495, 4503, 4508, 4511, 4519, 4527, 4535, 4540,
        4543, 4544, 4551, 4559, 4567, 4572, 4575, 4583, 4591, 4592,
        4599, 4604, 4607, 4615, 4623, 4631, 4636, 4639, 4647, 4655,
        4663, 4668, 4671, 4679, 4687, 4695, 4700, 4703, 4711, 4719,
        4720, 4727, 4732, 4735, 4743, 4751, 4759, 4764, 4767, 4775,
        4783, 4791, 4796, 4799, 4807, 4815, 4823, 4828, 4831, 4839,
        4847, 4848, 4855, 4860, 4863, 4871, 4879, 4887, 4892, 4895,
        4903, 4911, 4919, 4924, 4927, 4935, 4943, 4951, 4956, 4959,
        4967, 4975, 4976, 4983, 4988, 4991, 4999, 5007, 5015, 5020,
        5023, 5031, 5039, 5047, 5052, 5055, 5056, 5063, 5071, 5079,
        5084, 5087, 5095, 5103, 5104, 5111, 5116, 5119, 5127, 5135,
        5143, 5148, 5151, 5159, 5167, 5175, 5180, 5183, 5191, 5199,
        5207, 5212, 5215, 5223, 5231, 5232, 5239, 5244, 5247, 5255,
        5263, 5271, 5276, 5279, 5287, 5295, 5303, 5308, 5311, 5319,
        5327, 5335, 5340, 5343, 5351, 5359, 5360, 5367, 5372, 5375,
        5383, 5391, 5399, 5404, 5407, 5415, 5423, 5431, 5436, 5439,
        5447, 5455, 5463, 5468, 5471, 5479, 5487, 5488, 5495, 5500,
        5503, 5511, 5519, 5527, 5532, 5535, 5543, 5551, 5559, 5564,
        5567, 5568, 5575, 5583, 5591, 5596, 5599, 5607, 5615, 5616,
        5623, 5628, 5631, 5639, 5647, 5655, 5660, 5663, 5671, 5679,
        5687, 5692, 5695, 5703, 5711, 5719, 5724, 5727, 5735, 5743,
        5744, 5751, 5756, 5759, 5767, 5775, 5783, 5788, 5791, 5799,
        5807, 5815, 5820, 5823, 5831, 5839, 5847, 5852, 5855, 5863,
        5871, 5872, 5879, 5884, 5887, 5888, 5895, 5903, 5911, 5916,
        5919, 5927, 5935, 5943, 5948, 5951, 5959, 5967, 5975, 5980,
        5983, 5991, 5999, 6000, 6007, 6012, 6015, 6023, 6031, 6039,
        6044, 6047, 6055, 6063, 6071, 6076, 6079, 6080, 6087, 6095,
        6103, 6108, 6111, 6119, 6127, 6128, 6135, 6140, 6143, 6151,
        6159, 6167, 6172, 6175, 6183, 6191, 6199, 6204, 6207, 6215,
        6223, 6231, 6236, 6239, 6247, 6255, 6256, 6263, 6268, 6271,
        6279, 6287, 6295, 6300, 6303, 6311, 6319, 6327, 6332, 6335,
        6343, 6351, 6359, 6364, 6367, 6375, 6383, 6384, 6391, 6396,
        6399, 6407, 6415, 6423, 6428, 6431, 6439, 6447, 6455, 6460,
        6463, 6471, 6479, 6487, 6492, 6495, 6503, 6511, 6512, 6519,
        6524, 6527, 6535, 6543, 6551, 6556, 6559, 6567, 6575, 6583,
        6588, 6591, 6592, 6599, 6607, 6615, 6620, 6623, 6631, 6639,
        6640, 6647, 6652, 6655, 6663, 6671, 6679, 6684, 6687, 6695,
        6703, 6711, 6716, 6719, 6727, 6735, 6743, 6748, 6751, 6759,
        6767, 6768, 6775, 6780, 6783, 6791, 6799, 6807, 6812, 6815,
        6823, 6831, 6839, 6844, 6847, 6855, 6863, 6871, 6876, 6879,
        6887, 6895, 6896, 6903, 6908, 6911, 6919, 6927, 6935, 6940,
        6943, 6951, 6959, 6967, 6972, 6975, 6983, 6991, 6999, 7004,
        7007, 7015, 7023, 7024, 7031, 7036, 7039, 7047, 7055, 7063,
        7068, 7071, 7079, 7087, 7095, 7100, 7103, 7104, 7111, 7119,
        7127, 7132, 7135, 7143, 7151, 7152, 7159, 7164, 7167, 7168,
        7175, 7183, 7191, 7196, 7199, 7207, 7215, 7223, 7228, 7231,
        7239, 7247, 7255, 7260, 7263, 7271, 7279, 7280, 7287, 7292,
        7295, 7303, 7311, 7319, 7324, 7327, 7335, 7343, 7351, 7356,
        7359, 7367, 7375, 7383, 7388, 7391, 7399, 7407, 7408, 7415,
        7420, 7423, 7431, 7439, 7447, 7452, 7455, 7463, 7471, 7479,
        7484, 7487, 7495, 7503, 7511, 7516, 7519, 7527, 7535, 7536,
        7543, 7548, 7551, 7559, 7567, 7575, 7580, 7583, 7591, 7599,
        7607, 7612, 7615, 7616, 7623, 7631, 7639, 7644, 7647, 7655,
        7663, 7664, 7671, 7676, 7679, 7687, 7695, 7703, 7708, 7711,
        7719, 7727, 7735, 7740, 7743, 7751, 7759, 7767, 7772, 7775,
        7783, 7791, 7792, 7799, 7804, 7807, 7815, 7823, 7831, 7836,
        7839, 7847, 7855, 7863, 7868, 7871, 7879, 7887, 7895, 7900,
        7903, 7911, 7919, 7920, 7927, 7932, 7935, 7936, 7943, 7951,
        7959, 7964, 7967, 7975, 7983, 7991, 7996, 7999, 8007, 8015,
        8023, 8028, 8031, 8039, 8047, 8048, 8055, 8060, 8063, 8071,
        8079, 8087, 8092, 8095, 8103, 8111, 8119, 8124, 8127, 8128,
        8135, 8143, 8151, 8156, 8159, 8167, 8175, 8176, 8183, 8188,
        8191, 8199, 8207, 8215, 8220, 8223, 8231, 8239, 8247, 8252,
        8255, 8263, 8271, 8279, 8284, 8287, 8295, 8303, 8304, 8311,
        8316, 8319, 8327, 8335, 8343, 8348, 8351, 8359, 8367, 8375,
        8380, 8383, 8391, 8399, 8407, 8412, 8415, 8423, 8431, 8432,
        8439, 8444, 8447, 8455, 8463, 8471, 8476, 8479, 8487, 8495,
        8503, 8508, 8511, 8519, 8527, 8535, 8540, 8543, 8551, 8559,
        8560, 8567, 8572, 8575, 8583, 8591, 8599, 8604, 8607, 8615,
        8623, 8631, 8636, 8639, 8640, 8647, 8655, 8663, 8668, 8671,
        8679, 8687, 8688, 8695, 8700, 8703, 8711, 8719, 8727, 8732,
        8735, 8743, 8751, 8759, 8764, 8767, 8775, 8783, 8791, 8796,
        8799, 8807, 8815, 8816, 8823, 8828, 8831, 8839, 8847, 8855,
        8860, 8863, 8871, 8879, 8887, 8892, 8895, 8903, 8911, 8919,
        8924, 8927, 8935, 8943, 8944, 8951, 8956, 8959, 8967, 8975,
        8983, 8988, 8991, 8999, 9007, 9015, 9020, 9023, 9031, 9039,
        9047, 9052, 9055, 9063, 9071, 9072, 9079, 9084, 9087, 9095,
        9103, 9111, 9116, 9119, 9127, 9135, 9143, 9148, 9151, 9152,
        9159, 9167, 9175, 9180, 9183, 9191, 9199, 9200, 9207, 9212,
        9215, 9223, 9231, 9239, 9244, 9247, 9255, 9263, 9271, 9276,
        9279, 9287, 9295, 9303, 9308, 9311, 9319, 9327, 9328, 9335,
        9340, 9343, 9351, 9359, 9367, 9372, 9375, 9383, 9391, 9399,
        9404, 9407, 9415, 9423, 9431, 9436, 9439, 9447, 9455, 9456,
        9463, 9468, 9471, 9479, 9487, 9495, 9500, 9503, 9511, 9519,
        9527, 9532, 9535, 9543, 9551, 9559, 9564, 9567, 9575, 9583,
        9584, 9591, 9596, 9599, 9607, 9615, 9623, 9628, 9631, 9639,
        9647, 9655, 9660, 9663, 9664, 9671, 9679, 9687, 9692, 9695,
        9703, 9711, 9712, 9719, 9724, 9727, 9735, 9743, 9751, 9756,
        9759, 9767, 9775, 9783, 9788, 9791, 9799, 9807, 9815, 9820,
        9823, 9831, 9839, 9840, 9847, 9852, 9855, 9863, 9871, 9879,
        9884, 9887, 9895, 9903, 9911, 9916, 9919, 9927, 9935, 9943,
        9948, 9951, 9959, 9967, 9968, 9975, 9980, 9983, 9984, 9991,
        9999])

    if n in four_squares:
        return 4

    if n in two_squares:
        return 2

    # since every integer is representable as the sum of at
    # most four squares, and we've tried 1, 2, and 4...
    return 3
```
