https://stackoverflow.com/questions/15877914/java-overriding-equals-and-hashcode-for-two-interchangeable-integers

This is widely accepted approach:

    @Override
    public int hashCode() {
        int res = 17;
        res = res * 31 + Math.min(from, to);
        res = res * 31 + Math.max(from, to);
        return res;
    }