# Extended-Precision-Formatting-of-Doubles
This is a ToString like method will get the a closer approximation of the actual number as a string then .Net.  The rounding .net does is usually better but sometimes there is a need to view the additional percision.

# Testing
I did do some testing on this but not extended testing.

# The Function
    public static string ToStringFull(double value)
    {
        if (value == 0.0) return "0.0";
        if (double.IsNaN(value)) return "NaN";
        if (double.IsNegativeInfinity(value)) return "-Inf";
        if (double.IsPositiveInfinity(value)) return "+Inf";

        long bits = BitConverter.DoubleToInt64Bits(value);
        BigInteger mantissa = (bits & 0xfffffffffffffL) | 0x10000000000000L;
        int exp = (int)((bits >> 52) & 0x7ffL) - 1023;
        string sign = (value < 0) ? "-" : "";

        if (54 > exp)
        {
            double offset = (exp / 3.321928094887362358); //...or =Math.Log10(Math.Abs(value))
            BigInteger temp = mantissa * BigInteger.Pow(10, 26 - (int)offset) >> (52 - exp);
            string numberText = temp.ToString();
            int digitsNeeded = (int)((numberText[0] - '5') / 10.0 - offset);
            if (exp < 0)
                return sign + "0." + new string('0', digitsNeeded) + numberText;
            else
                return sign + numberText.Insert(1 - digitsNeeded, ".");
        }
        return sign + (mantissa >> (52 - exp)).ToString();
    }
    
# Example Code and Output
	// Just as the mantissa rolls over
	ToStringFull(0.00000000000000044408920985006100)                    --> 0.00000000000000044408920985006098914383564
	ToStringFull(0.00000000000000044408920985006257)                    --> 0.00000000000000044408920985006256686564609
	ToStringFull(0.00000000000000044408920985006400)                    --> 0.00000000000000044408920985006399667603680
	ToStringFull(BitConverter.Int64BitsToDouble(0x4400000000000000)))   --> 36893488147419103232
	ToStringFull(BitConverter.Int64BitsToDouble(0x43FFFFFFFFFFFFFF)))   --> 36893488147419099136

	// Just as the Base10 rolls over
	ToStringFull(0.0000000000000009999999999998) --> 0.00000000000000099999999999979990425070004
	ToStringFull(0.0000000000000009999999999999) --> 0.00000000000000099999999999990008958566311
	ToStringFull(0.0000000000000010000000000000) --> 0.00000000000000100000000000000007770539987
	ToStringFull(0.0000000000000010000000000001) --> 0.00000000000000100000000000010006582513663

	// Special Numbers
	ToStringFull(0.0)                     -->  0.0
	ToStringFull(double.NaN)              -->  NaN 
	ToStringFull(double.Epsilon)          -->  0.000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000011125369292536009385779392 [FAILS - should be 5E-324]
	ToStringFull(-double.Epsilon)         --> -0.000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000011125369292536009385779392 [FAILS - should be -5E-324]
	ToStringFull(double.MaxValue)         -->  179769313486231570814527423731704356798070567525844996598917476803157260780028538760589558632766878171540458953514382464234321326889464182768467546703537516986049910576551282076245490090389328944075868508455133942304583236903222948165808559332123348274797826204144723168738177180919299881250404026184124858368
	ToStringFull(double.MinValue)         --> -179769313486231570814527423731704356798070567525844996598917476803157260780028538760589558632766878171540458953514382464234321326889464182768467546703537516986049910576551282076245490090389328944075868508455133942304583236903222948165808559332123348274797826204144723168738177180919299881250404026184124858368
	ToStringFull(double.NegativeInfinity) --> -Inf
	ToStringFull(double.PositiveInfinity) --> Inf

	// Large range of sizes
	ToStringFull(0.00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000123456789)
		  -->    0.00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000123456788999999994258263493
	
	ToStringFull(0.00000000000123)  --> 0.00000000000123000000000000000959180317
	ToStringFull(0.0000000000123)   --> 0.0000000000123000000000000004998148152
	ToStringFull(0.000000000123)    --> 0.00000000012299999999999998884227681
	ToStringFull(0.00000000123)     --> 0.00000000123000000000000009521792127
	ToStringFull(0.0000000123)      --> 0.0000000122999999999999992978179876
	ToStringFull(0.000000123)       --> 0.00000012300000000000000290434722
	ToStringFull(0.00000123)        --> 0.00000123000000000000008198303147
	ToStringFull(0.0000123)         --> 0.0000123000000000000008198303147
	ToStringFull(0.000123)          --> 0.00012300000000000000819830314
	ToStringFull(0.00123)           --> 0.00122999999999999997356281422
	ToStringFull(0.0123)            --> 0.0123000000000000001693090112
	ToStringFull(0.123)             --> 0.122999999999999998223643160
	ToStringFull(1.23)              --> 1.22999999999999998223643160
	ToStringFull(12.3)              --> 12.30000000000000071054273576
	ToStringFull(123.0)             --> 123.0000000000000000000000000
	ToStringFull(1230.0)            --> 1230.00000000000000000000000
	ToStringFull(12300.0)           --> 12300.00000000000000000000000
	ToStringFull(123000.0)          --> 123000.0000000000000000000000
	ToStringFull(1230000.0)         --> 1230000.00000000000000000000
	ToStringFull(12300000.0)        --> 12300000.00000000000000000000
	ToStringFull(123000000.0)       --> 123000000.0000000000000000000
	ToStringFull(1230000000.0)      --> 1230000000.00000000000000000
	ToStringFull(12300000000.0)     --> 12300000000.00000000000000000
	ToStringFull(123000000000.0)    --> 123000000000.0000000000000000
	ToStringFull(1230000000000.0)   --> 1230000000000.00000000000000
	ToStringFull(12300000000000000000000000000000000000000.0)  --> 12299999999999999515319483038827796234240

	\\ Large range of sizes 
	ToStringFull(12300000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000.0)
			 --> 12299999999999998771969295666294362108999337860319719278427394074913328075878379011501919067577412170076715635800056490126571748024385536
	ToStringFull(-12300000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000.0)
			--> -12299999999999998771969295666294362108999337860319719278427394074913328075878379011501919067577412170076715635800056490126571748024385536
			
	\\ Other
	double i = (10 * 0.69);
	ToStringFull(i)       --> 6.89999999999999946709294817
	ToStringFull(-6.9)    --> -6.90000000000000035527136788
	ToStringFull(i - 6.9) --> -0.00000000000000088817841970012523233890533
		


