import java.io.*;
import java.util.*;

/**
 * Lab 1 - Two-pass Linker
 * Operating Systems, Spring 2016
 * Sean Basler
 */
public class Linker {
    static Map<String, Integer> symbolTable = new HashMap<String, Integer>();
    static ArrayList<Integer> memoryMap = new ArrayList<Integer>();
    static ArrayList<Integer> baseAddressOfModule = new ArrayList<Integer>();
    static int numberOfModules = 0;
    
    /**
     * New scanner to take in buffered input
     * @param file
     * @return
     */
    public static Scanner myScanner(String file) {
        try {
            Scanner input = new Scanner(new BufferedReader(new FileReader(file)));
            return input;
        }
        catch(Exception e) {
            System.out.printf("Can't read %s\n", file);
            e.printStackTrace(System.out);
            System.exit(0);
        }
        return null;
    }
    
    /**
     * Pass one, which creates the symbol table and stores base addresses, then prints
     * @param input
     */
    public static void passOne(Scanner input) {
        ArrayList<String> moduleSymbols = new ArrayList<String>();
        ArrayList<Integer> moduleAddresses = new ArrayList<Integer>();
        
        ArrayList<String> symbolList = new ArrayList<String>();
        ArrayList<String> useList = new ArrayList<String>();
        
        int numberOfLines = 0;
        int numberOfItemsInLine;
        int addressLineNumber = 0;
  
        int moduleNumber = -1;
        int absoluteAddress = 0;
        
        //goes through entire raw input, which conveniently alternates string/int
        while (input.hasNext()) {
            numberOfLines++;
            numberOfItemsInLine = input.nextInt();
            
            String symbol;
            int wordAddress;
            
            if ((numberOfLines - 1) % 3 == 0) {
                baseAddressOfModule.add(addressLineNumber);
                moduleSymbols.clear();
                moduleAddresses.clear();
                moduleNumber++;
            }
            
            //goes through each line, taking the appropriate action depending on whether it's a definition, use, or address
            for (int i = 0; i < numberOfItemsInLine; i++) {
                symbol = input.next();
                wordAddress = input.nextInt();
                
                //takes in the symbols (if any)
                if ((numberOfLines - 1) % 3 == 0) {
                    if (!symbolTable.containsKey(symbol)) {
                        absoluteAddress = wordAddress + baseAddressOfModule.get(moduleNumber);
                        symbolTable.put(symbol, absoluteAddress);
                        symbolList.add(symbol);
                        moduleSymbols.add(symbol);
                        moduleAddresses.add(wordAddress);
                    }
                    else {
                        System.out.printf("Error: " + symbol + " is multiply defined; first value used.\n");
                    }  
                }
                
                //takes in uses
                if ((numberOfLines - 2) % 3 == 0) {
                    if (!useList.contains(symbol)) {
                        useList.add(symbol);
                    }    
                }
                
                //takes in program text
                if (numberOfLines % 3 == 0) {
                    addressLineNumber++;

                    for (int j = 0; j < moduleSymbols.size(); j++) {
                        if (moduleAddresses.get(j) >= numberOfItemsInLine) {
                            System.out.printf("Error: The value of %s is outside module %d; zero (relative) used.\n", 
                                moduleSymbols.get(j) , moduleNumber + 1);
                            symbolTable.put(moduleSymbols.get(j), baseAddressOfModule.get(moduleNumber));
                        }
                    }
                }
            }
        }
        
        //after going through all input, checks to see if an item is defined but not used
        for (int i = 0; i < symbolList.size(); i++) {
            if (!useList.contains(symbolList.get(i))) {
                System.out.printf("Warning: %s is defined but never used.\n", symbolList.get(i));
            }    
        }
        System.out.println();

        System.out.println("Symbol Table");
        for (Map.Entry<?,?> entry : symbolTable.entrySet()) {
            System.out.printf("%s=%d\n", entry.getKey(), entry.getValue());
        }
    }
    
    /**
     * Pass two, which relocates relative addresses and resolves external references, then prints
     * @param input
     */
    public static void passTwo(Scanner input) {
        ArrayList<String> symbolList = new ArrayList<String>();
        ArrayList<Comparable> useList = new ArrayList<Comparable>();
        ArrayList<Integer> programText = new ArrayList<Integer>();

        int numberOfLines = 0;
        int numberOfItemsInLine;
      
        while (input.hasNext()) {
            numberOfLines++;
            numberOfItemsInLine = input.nextInt();
            
            String symbol;
            int wordAddress;
            
            for (int i = 0; i < numberOfItemsInLine; i++) {
                symbol = input.next();
                wordAddress = input.nextInt();
                
                if ((numberOfLines - 2) % 3 == 0) {
                    if (symbolTable.containsKey(symbol)) {
                        useList.add(symbol);
                        useList.add(wordAddress);
                    } else {
                        useList.add(symbol);
                        System.out.printf("Error: " + symbol + " used but not defined; zero used.\n");
                        useList.add(0);
                    }
                }
                
                if (numberOfLines % 3 == 0) {
                    symbolList.add(symbol);
                    programText.add(wordAddress);
                }
            }
            
            if (numberOfLines % 3 == 0) {
                memoryMap = combineProgramSymbolUseLists(memoryMap, symbolList, useList, programText);                      
                symbolList.clear();
                useList.clear();
                programText.clear();              
            }
        }
        
        //prints out the Memory Map
        System.out.println();
        System.out.println("Memory Map");
        for (int i = 0; i < memoryMap.size(); i++) {
            System.out.printf("%-4d:  %d\n", i, memoryMap.get(i));
        }
    }
    
    /**
     * Combines lists, resolves external used addresses, relocates relative addresses, and resolves external used addresses
     * @param combinedList
     * @param useList
     * @param programList
     * @param symbolList
     * @return
     */
    public static ArrayList<Integer> combineProgramSymbolUseLists(ArrayList<Integer> combinedList, 
        ArrayList<String> symbolList, ArrayList<Comparable> useList, ArrayList<Integer> programList) {
        
        Integer address;
        Integer number;
        
        for (int i = 0; i < useList.size(); i += 2) {
            number = symbolTable.get(useList.get(i));
            address = (Integer) useList.get(i + 1);
            address = resolveRelativeAddresses(programList, symbolList, address);
            
            if (number != null) {
                number = ((programList.get(address) / 1000) * 1000) + number;
            } else {
                number = ((programList.get(address) / 1000) * 1000);
            }    
            programList.set(address, number);
        }
        
        for (int i = 0; i < symbolList.size(); i++) {
            if (symbolList.get(i).equals("USED")) {
                address = resolveRelativeAddresses(programList, symbolList, i);
                int addressValue = programList.get(i);
                int relativeValue = programList.get(address);
                number = 1000 * (addressValue / 1000) + (relativeValue % 1000);
                programList.set(i, number);
            } else if (symbolList.get(i).equals("R")) {
                number = (programList.get(i) % 1000) + baseAddressOfModule.get(numberOfModules) 
                    + (programList.get(i) / 1000 * 1000);
                programList.set(i, number);
            }
        }
        
        for (int i = 0; i < symbolList.size(); i++) {
            if (symbolList.get(i).equals("E")) {
                System.out.printf("Error: E type address not on use chain; treated as I type.\n");
            }
        }
        
        combinedList.addAll(programList);
        numberOfModules++;
        return combinedList;
    }
    
    /**
     * Returns the relocated relative address
     * @param programList
     * @param symbolList
     * @param location
     * @return
     */
    public static int resolveRelativeAddresses(ArrayList<Integer> programList, 
        ArrayList<String> symbolList, int location) {
        int followingAddress = programList.get(location);
        
        if (!symbolList.get(location % 1000).equals("E") && !symbolList.get(location % 1000).equals("DONE") &&
                !symbolList.get(location % 1000).equals("USED")) {
            System.out.printf("Error: %s type address on use chain; treated as E type.\n", 
                symbolList.get(location % 1000));
        }
        
        if (followingAddress % 1000 == 777) {
            symbolList.set(location, "DONE");
            return location;
        } else if (symbolList.get(location).equals("DONE")) {
            symbolList.set(location, "DONE");
            return location;
        } else if ((followingAddress % 1000) > symbolList.size()) {
            symbolList.set(location, "DONE");
            System.out.printf("Error: Pointer in use chain exceeds module size; chain terminated.\n", 
                followingAddress, symbolList.size());
            return location;
        }
        
        else if (symbolList.get(followingAddress % 1000).equals("DONE")) {
            symbolList.set(location, "DONE");
            int addressValue = programList.get(location);
            int relativeValue = programList.get(followingAddress % 1000);
            int number = 1000 * (addressValue / 1000) + (relativeValue % 1000);
            programList.set(location, number);
            return followingAddress % 1000;
        }
        symbolList.set(location, "USED");
        
        return resolveRelativeAddresses(programList, symbolList, followingAddress % 1000);
    }
    
    /**
     * Main program where user can specify input file
     * @param args
     */
    public static void main (String [] args) {
        System.out.println("Which file would you like to test?");
        
        Scanner scanner = new Scanner(System.in);
        String file = scanner.nextLine();
        scanner.close();
        Scanner input = myScanner(file);
        
        passOne(input);
        input = myScanner(file);
        passTwo(input);
        input.close();
    }
}
